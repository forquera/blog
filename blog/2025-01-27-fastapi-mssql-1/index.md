---
slug: fastapi-mssql-1
title: Conectando FastAPI a una base de datos SQL Server existente - Parte 1
authors: [orquera]
tags: [Python, FastAPI, SQL Server]
---

Hola!, vamos a ver como conectar un proyecto FastAPI a una base de datos existente SQL Server desde Windows.

<!-- truncate -->

Lo primero que vamos a hacer es crear un proyecto en FastAPI. Lo vamos a instalar a través del siguiente comando:

```shell showLineNumbers
pip install "fastapi[standard]"
```

Una vez instalado, dentro del proyecto creo una carpeta llamada database en donde guardare las configuraciones relacionadas a la base de dato.

Para conectarse a una base de datos específica, se necesita de un driver para establecer la conexión, el driver es una capa de traducción entre la aplicación y la base de datos. En este caso al ser SQL Server usare el driver ODBC Driver 18.

En el siguiente enlace se puede descargar el driver para instalar [ODBC Driver 18](https://learn.microsoft.com/es-es/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver16).

Luego tenemos que instalar la librería en python para poder usar el driver.

```shell showLineNumbers
pip install pyodbc
```

Luego procedo a crear un archivo config.py dentro de la carpeta database en donde estará la configuración para poder conectarme.

```python showLineNumbers title="/database/config.py"
class Settings:
    def __init__(self):
        self.__server = "192.168.0.10"
        self.__port = "1433"
        self.__database = "databasenombre"
        self.__username = "prueba"
        self.__password = "prueba2025"
        self.__database_url = f"mssql+pyodbc://{self.__username}:{self.__password}@{self.__server}:{self.__port}/{self.__database}?driver=ODBC+Driver+18+for+SQL+Server&TrustServerCertificate=yes"

    @property
    def database_url(self):
        return self.__database_url


settings = Settings()
```

Una vez que ya tenemos la configuración para establecer la conexión con la base de datos. En FastAPI se trabaja con una "Session", que se encarga de abrir la conexión, realizar la consulta y de cerrar la conexión. Vamos a crear otro archivo dentro de la carpeta database para crear la "Session" y de ahí utilizarla en otros lados.

```python showLineNumbers title="/database/session.py"
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from database.config import settings

# Configuracion de la base de datos.
SQLALCHEMY_DATABASE_URL = settings.database_url

# Crear el engine
engine = create_engine(SQLALCHEMY_DATABASE_URL)

# Crear una sesion para interactuar con la base de datos.
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Dependencia para obtener sesiones de base de datos.
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

Lo que estamos haciendo es:

- Con la funcion _create_engine()_ lo que hacemos es crear un puente entre la aplicación y la base de datos. Y se va a encargar de ejecutar las consultas y de manejar el pool de conexiones, este trabaja a bajo nivel y se crea una vez al inicio de la aplicacíon.
- Con _sessionmaker()_ creamos sesiones que interactúan con el _engine_, con las cuales vamos a realizar las consultas a la base de datos. Este trabaja a un alto nivel, permite trabajar usando modelos ORM y se crea una sesión por solicitud o tarea.

Y con esos dos archivos ya podemos realizar consultas a la base de datos, usando dependencia de FastAPI que lo veremos en la próxima publicación.
