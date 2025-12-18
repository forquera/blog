---
slug: otp-python
title: Creando un token de seguridad con python.
authors: [orquera]
tags: [Python, OTP, PyOTP]
---

Hola!, en la última publicación del año, vamos a ver cómo podemos crear un token de seguridad de 6 dígitos usando la librería PyOTP en Python.

<!-- truncate -->

[PyOTP](https://pyauth.github.io/pyotp/) es una biblioteca de Python para generar y verificar contraseñas de un solo uso. Permite implementar métodos de autenticación de dos factores (2FA) o multifactor (MFA) en aplicaciones web y otros sistemas que requieren el inicio de sesión de los usuarios.

Lo primero que vamos a hacer es instalar la librería PyOTP con el comando:

```shell showLineNumbers
pip install pyotp
```

Una vez instalado, podemos importar la librería.

```python showLineNumbers title="main.py"
import pyotp
```

Luego vamos a instanciar un objeto del tipo TOTP, el cual necesita que le pasemos por parámetro un secreto codificado en Base32. También como opcional se le puede pasar la cantidad de dígitos que queremos generar los tokens y el intervalo de tiempo en el que el token será valido, por defecto es de 30 segundos.

Luego usando el método now, genero un token para el tiempo actual.

```python showLineNumbers title="main.py"
import pyotp
"""
    Aquí instancio un objeto y le indico que quiero token
    de 6 dígitos  con una validez de 60 segundos.
"""
totp = pyotp.TOTP(secret_base_32, digits=6, interval=60)

"""
    Aquí genero un token vigente.
"""
token = totp.now() # => "316439"
```

Luego para verificar si un token es valido o no, puedo utilizar el metodo verify. Y opcionalmente le puedo pasar la tolerancia de tiempo que tendre para validar el token, es decir, cuantas ventanas de tiempo, aparte de la actual, se consideraran válidas

```python showLineNumbers title="main.py"
import pyotp
"""
    Aquí instancio un objeto y le indico que quiero token
    de 6 dígitos  con una validez de 60 segundos.
"""
totp = pyotp.TOTP(secret_base_32, digits=6, interval=60)

"""
    Aquí genero un token vigente.
"""
token = totp.now() # => "316439"

"""
    Aquí valido un token. Y le indico que tenga tolerancia
    una ventana hacia atrás y otra hacia adelante de la actual.
    Por lo que se aceptaran tokens de 60 segundos anteriores y
    60 segundos posteriores, aparte de la ventana actual.
"""
totp.verify(token, valid_window=1) # => True
```

Y de esta forma sencilla podemos generar tokens y validarlos. Para poder usarlos en diferentes aplicaciones que requieran un password de un solo uso.

¡Hasta la próxima y felices fiestas! 🎉
