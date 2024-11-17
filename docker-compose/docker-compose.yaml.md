! Check on enviroment files.
! Create dir for composing.

```
version: '3.8'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    env_file:
      - .env.db  # Load environment variables from the .env.db file
    networks:
      - custom_network

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "${WORDPRESS_PORT}:80"  # Port mapping from .env.wordpress file
    restart: always
    env_file:
      - .env.wordpress  # Load environment variables from the .env.wordpress file
    networks:
      - custom_network

volumes:
  db_data:
  wordpress_data:

networks:
  custom_network:
    driver: bridge
```
