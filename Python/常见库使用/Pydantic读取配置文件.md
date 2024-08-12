
# Pydantic 是什么？
Pydantic 是一个用于数据验证和配置管理的 Python 库。它通过 Python 类型注解来定义数据模型，并提供了强大的数据验证功能。Pydantic 的主要目标是确保数据在输入和输出时的一致性和有效性

## pydantic 安装（笔记中使用的pydantic版本是2.7.1 pydantic-settings的版本是2.2.1）
~~~shell
pip install pydantic pydantic-settings
~~~

## Pydantic读取配置文件
### config.yml配置文件
~~~yml
mysql:
  host: localhost
  port: 3306
  db: testdb
  user: root
  password: 123456
~~~
### 读取配置文件
~~~python
from typing import Type, Tuple

from pydantic import BaseModel, computed_field, MySQLDsn
from pydantic_settings import (
    BaseSettings,
    SettingsConfigDict,
    PydanticBaseSettingsSource,
    YamlConfigSettingsSource,
)

# 配置实体类需要继承BaseModel
# BaseModel的作用是提供数据验证、数据解析、数据转换等功能将输入的数据（从环境变量或者配置文件）转换为配置实体对象实例
class MySQL(BaseModel):
    host: str = 'localhost'
    port: int = 3306
    user: str = 'root'
    password: str = '123456'
    db: str = 'testdb'

class Settings(BaseSettings):
    mysql: MySQL

    # @computed_field注解的装饰器用于定义基于其他字段计算得来的属性
    # 这些计算属性不会被序列化或存储在模型的原始数据中，但会在访问时动态计算并返回
    @computed_field
    def mysql_dsn(self) -> MySQLDsn:
        return f"mysql+mysqldb://{self.mysql.user}:{self.mysql.password}@{self.mysql.host}:{self.mysql.port}/{self.mysql.db}"

    # 指定多个配置文件，注意这是当前程序的运行路径
    model_config = SettingsConfigDict(
        yaml_file=[
            "config.yml",
            "config.dev.yml",
            "config.staging.yml",
            "config.prod.yml",
            "config.local.yml",
        ],
        yaml_file_encoding="utf-8",
        env_file=[".env", ".env.dev", ".env.staging", ".env.prod", ".env.local"],
        env_file_encoding="utf-8",
        extra="ignore",
    )

    @classmethod
    def settings_customise_sources(
        cls,
        settings_cls: Type[BaseSettings],
        init_settings: PydanticBaseSettingsSource,
        env_settings: PydanticBaseSettingsSource,
        dotenv_settings: PydanticBaseSettingsSource,
        file_secret_settings: PydanticBaseSettingsSource,
    ) -> Tuple[PydanticBaseSettingsSource, ...]:
        return (
            init_settings,
            env_settings,
            dotenv_settings,
            YamlConfigSettingsSource(settings_cls),
            file_secret_settings,
        )


# 初始化加载配置文件转换为配置实体对象实例
settings = Settings()

if __name__ == "__main__":
    print(settings)
    print(settings.mysql)
    print(settings.mysql_dsn)

~~~