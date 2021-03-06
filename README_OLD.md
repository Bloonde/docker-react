# Container con aplicación React

Instrucciones para crear un contenedor con una aplicación React construida con create-react-app conectada a un directorio de desarrollo en nuestra máquina con hot-reloading.

Requisitos:

- [Docker](#Docker)

- [React](#React)



## Docker

### Linux (Debian/Ubuntu)

1. Instalar [Docker](https://www.digitalocean.com/community/tutorials/como-instalar-y-usar-docker-en-ubuntu-18-04-1-es) desde Terminal.
2. Verificar instalación ejecutando `docker run hello-world` en Terminal.

[Documentación oficial](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

### Mac

1. Descargar e instalar [Docker Desktop](https://hub.docker.com/?overlay=onboarding).
2. Verificar instalación ejecutando `docker run hello-world` en Terminal.

[Documentación oficial](https://docs.docker.com/docker-for-mac/install/)

### Windows (Home)

Windows Home no viene con hypervisor nativo, vamos a necesitar VirtualBox para poder usar Docker.

1. Asegurarse de que Windows es 64-bit y que la virtualización está activada en la BIOS.
2. Descargar e instalar la última versión de [Docker Toolbox](https://github.com/docker/toolbox/releases), que contiene Docker QuickStart Terminal, Kitematic y Oracle VM VirtualBox.
3. Verificar instalación abriendo Docker QuickStart Terminal y ejecutando `docker run hello-world`. El terminal de Docker usa Bash, **no usar CMD nativo de Windows**, que no soporta comandos estándar de Unix necesarios para trabajar con Docker.

[Documentación oficial](https://docs.docker.com/toolbox/toolbox_install_windows/)

### Windows (Pro/Enterprise)

1. Asegurarse de que [Hyper-V y Windows Containers](https://docs.microsoft.com/es-es/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) están activados.
2. Descargar e instalar [Docker Desktop](https://hub.docker.com/?overlay=onboarding).
3. Verificar instalación ejecutando `docker run hello-world` en PowerShell.

[Documentación oficial](https://docs.docker.com/docker-for-windows/install/)



## React

1. Se necesita Node y npm (atención a las versiones y su compatibilidad con create-react-app).
2. Crear proyecto en nuestro directorio de desarrollo: `npx create-react-app react-app`
   Ahora ya no se instala create-react-app globalmente y puede dar error de instalación si ya estaba instalado. Hay que desinstalarlo con `npm uninstall -g create-react-app`, buscar si queda algo con `which create-react-app` y borrar el directorio que salga `rm -rf /usr/local/bin/create-react-app`
3. Acceder al proyecto: `cd react-app`

[Documentación oficial](https://create-react-app.dev/)



## Container de desarrollo

Desde raíz del proyecto (`pwd`):

1. Añadir un archivo Dockerfile (sin extensión y con D mayúscula) con el editor que queráis (ejemplo `nano Dockerfile`) con el siguiente contenido:

   ```
   # imagen base con Node en un Linux Alpine
   FROM node:12.2.0-alpine
   
   # directorio de trabajo en el contenedor
   WORKDIR /app
   
   # añadir `/app/node_modules/.bin` a $PATH
   ENV PATH /app/node_modules/.bin:$PATH
   
   # copiar configuración e instalar dependencias
   COPY package.json /app/package.json
   RUN npm install
   
   # ejecutar servidor de desarrollo
   CMD ["npm", "start"]
   ```


2. Añadir un archivo .dockerignore para que Docker ignore los directorios git y node_modules y agilizar el proceso posterior de levantar el contenedor:

   ```
   node_modules
   .git
   .gitignore
   ```

3. Construir la imagen: `docker build -t react-app:dev .`

4. Verificar que la imagen existe: `docker images` (en el listado debe aparecer una imagen react-app con el tag dev).

5. Crear un contenedor a partir de la imagen con las siguientes opciones:

   - Dos volúmenes: uno en `/app` unido al directorio raíz del proyecto (${PWD}) y otro anónimo `/app/node_modules` que hace que la aplicación tire de node_modules de dentro del contenedor que ha construido en la imagen y no lo sobreescriba de nuevo durante el run.
   - Dos puertos: 3000 para interconexión con otros contenedores en la misma red enlazado a 3001 como host para acceder desde nuestra máquina.
   - `—rm`elimina el contenedor y los volúmenes cuando se para el contenedor.
   - `:dev` es un flag para identificar que el contenedor está en modo desarrollo en el listado de `docker ps`, estamos haciendo `serve`, no `build`.
   - `angular-app:dev` es la imagen con la que vamos a levantar el contenedor.

   ```
   docker run -v ${PWD}:/app -v /app/node_modules -p 3001:3000 --rm react-app:dev
   ```

6. Verificar abriendo [http://localhost:3001](http://localhost:3001) en el navegador y que con `docker ps` aparece un contenedor levantado con la imagen que hemos creado antes `react-app:dev`.

7. Para verificar el hot-reloading modifica un componente en la aplicación (`src/App.js` por ejemplo), el navegador debe recargarse con los cambios.

8. El proceso `run` se queda ejecutandose en la terminal (podemos añadir el flag -d para que se ejecute en background pero no recibes feedback) se puede parar con `ctrl+c` o abrir otro terminal y ejecutar `docker stop {container id}`



### Docker compose

Para no tener que acordarse de las opciones de run podemos usar docker-compose y meter toda la configuración en un archivo `docker-compose.yml` en raíz del proyecto:

```yaml
version: '3'

services:

  angular-app:
    container_name: react-app
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - '.:/app'
      - '/app/node_modules'
    ports:
      - '3001:3000'
    environment:
      - NODE_ENV=development

```

Ahora levantamos el contenedor con: `docker-compose up -d`. 

Hay que tener en cuenta que si usamos docker-compose va a crear la imagen a partir del Dockerfile, no necesitamos ejecutar `build` primero, y luego va a ejecutar `run` con las opciones de volúmenes y puertos igual que con el comando `run`.

Con `docker-compose` podemos añadir más servicios (contenedores) como bases de datos, servidor, app para backoffice, etc. e interconectarlos.

Para parar el contendor: `docker-compose stop`



## Aplicación prexistente

Para crear un contenedor desde una aplicación existente se sigue las mismas instrucciones pero adaptando las opciones de versiones a las de la aplicación (CLI, imagen base de Node, etc.), al fin y al cabo en este ejemplo también hemos creado el contenedor después de crear la aplicación.
