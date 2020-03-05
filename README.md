# Docker React

Create .env

    $ cd docker-react
    $ cp .env.example .env

Edit .env (Docker-react)

    -PROJECT_NAME=project_name
    -IMAGE_NAME=vue // Not edited

Start:

    $ cd docker-react
    $ ./start.sh
    
Run container and composer project

    $ docker exec -it project_name bash
    $ su docker
    
