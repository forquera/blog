---
slug: nginx-proxy-inverso
title: Usar Nginx como proxy inverso
authors: [orquera]
tags: [Nginx, Ubuntu, Server, Proxy]
---

Buenos días, buenas tardes y buenas noches. Vamos a ver cómo podemos utilizar Nginx como un proxy inverso en un servido Ubuntu.

<!-- truncate -->

Lo primero que tenemos que realizar es instalar Nginx en Ubuntu, para eso ejecutamos el comando en la terminal:

```bash
sudo apt install nginx -y
```

Para comprobar que se instaló y se está ejecutando, podemos ejecutar el comando:

```bash
sudo systemctl status nginx
```

Y debería salir "Active: active (running)".

Nginx se instala en **/etc/nginx** y contiene una estructura de carpetas para poder configurarlo.

- **/etc/nginx/nginx.conf**: el archivo principal de configuración.
- **/etc/nginx/sites-available/**: se usa para almacenar archivos de configuración para diferentes servidores virtuales. Cada archivo representa un sitio distinto, pero no activa los sitios.
- **/etc/nginx/sites-enabled/**: en esta carpeta hay enlaces simbólicos a los archivos de sites-available el cual indicaran a Nginx cual utilizar. Es una forma práctica de activar o desactivar sin tener que andar borrando los archivos de configuración.

A todo esto, supongamos que ya tenemos un dominio miapp.dominio.com y que apunta al servidor donde tenemos alojado Nginx, el cual recibir la petición.

Por lo que primero vamos a crear un archivo de configuración dentro de sites-available:

```bash
sudo nano /etc/nginx/sites-available/miapp
```

y dentro del archivo colocamos:

```nginx
server {
    listen 80;

    server_name miapp.dominio.com;

    location / {
                proxy_pass http://127.0.0.1:3500;
                proxy_set_header Host $host;
        };
}
```

Aquí lo que estamos definiendo es un servidor virtual:

- Con listen 80, le estamos indicando que escuche peticiones en el puerto 80, que son las peticiones HTTP.
- Con server_name miapp.dominio.com le indicamos que solo responda peticiones con dominio "miapp.dominio.com".
- Con location / {} definimos el comportamiento para todas las rutas "/", es decir, "miapp.dominio.com/", "miapp.dominio.com/about", "miapp.dominio.com/api/" van a pasar por este bloque de servidor.
- Con proxy_pass indicamos a donde debe redirigir la petición, puede ser una dirección IP, un dominio o el servidor back-end podría estar en el mismo server.
- Con proxy_set_header Host $host le indico que al redirigir la petición, mantenga el encabezado original. Para que el back-end vea la petición como si viniera del dominio original (miapp.dominio.com) en lugar de la IP de Nginx.

Una vez realizada la configuración, guardamos los cambios y tenemos que
crear un enlace simbólico en la carpeta **/etc/nginx/sites-enabled/**, podemos usar el comando:

```bash
sudo ln -s /etc/nginx/sites-available/miapp /etc/nginx/sites-enabled/
```

Y reiniciamos Nginx para que se vean afectados los cambios:

```bash
sudo systemctl restart nginx
```

Con lo que todas las peticiones hechas a nuestro dominio, deben ser redirigidas por Nginx. Pero nos está faltando algo, nos están faltando las peticiones HTTPS.

Supondremos que contamos con los certificados SSL. Volveremos a editar el archivo de configuración y modificaremos lo siguiente:

```nginx
server {
    listen 80;

    server_name miapp.dominio.com;

    return 301 https://$host$request_uri;
}

server{
        listen 443 ssl;
        server_name miapp.dominio.com;

        ssl_certificate /home/ssl/certificado.crt;
        ssl_certificate_key /home/ssl/privado/llave.key;

        location / {
                proxy_pass http://127.0.0.1:3500;
                proxy_set_header Host $host;
        }
}
```

- Con return 301 https://$host$request_uri; respondemos el estado Redirección Permanente y se redirige al mismo host y ruta, pero usando HTTPS.
- Agregamos otro servidor virtual el cual escuchara peticiones que sean HTTPS, el cual ingresan por el puerto 443.
- Con ssl_certificate y ssl_certificate_key indicamos en donde se encuentran nuestros certificados.

De esta manera le agregamos HTTPS y cualquier petición que ingrese con HTTP será redirigida HTTPS.

Reiniciamos Nginx:

```bash
sudo systemctl restart nginx
```

Y listo, de esta manera podemos configurar de una forma básica Nginx como proxy-inverso.
