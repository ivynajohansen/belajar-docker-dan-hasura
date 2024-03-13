# Pembuatan docker image dari program sederhana (REST API)
## 1. Node.js API
### Langkah 1: Buat proyek Node.js
Jalankan `npm init -y` untuk membuat file package.json default. 

![image](https://github.com/ivynajohansen/belajar-docker/assets/83331802/9be47c42-e870-4c66-87b6-82beb6256e02)

### Langkah 2: Buat file kode server

Setelah proyek Node.js dibuat, buat file yang akan berisi kode server Node.js:

`touch server.js`

Isi file dengan kode:

```
const http = require('http');
const mysql = require('mysql');

const connection = mysql.createConnection({
  host: 'mysql-container', // Use the container name
  user: 'mysql',
  password: 'welcomemysql',
  database: 'Product',
  authPlugin: 'caching_sha2_password'
});

connection.connect((err) => {
  if (err) {
    console.error('Error connecting to MySQL:', err.stack);
    return;
  }
  console.log('Connected to MySQL as id', connection.threadId);
});

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Langkah 2: Add Dockerfile

Di folder yang sama dengan docker-compose.yml dan server.js, gunakan perintah `touch Dockerfile` untuk membuat file Dockerfile, yang diisi dengan

```
FROM node:14

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

### Langkah 3: Add service API dan Network ke docker compose
Update docker-compose.yml untuk memasukkan service API Node.js, dan tambahkan network supaya container MySQL dan API bisa saling terhubung.

```
version: '3.3'

services:
  mysql:
    image: mysql:latest
    container_name: mysql-container
    restart: always
    environment:
      MYSQL_DATABASE: Product
      MYSQL_USER: mysql
      MYSQL_PASSWORD: welcomemysql
      MYSQL_ROOT_PASSWORD: welcomemysql
      MYSQL_AUTHENTICATION_PLUGIN: caching_sha2_password
    ports:
      - "3306:3306"
    networks:
      - node-network

  postgres:
    image: postgres:latest
    container_name: postgres-container
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: welcomepostgres
      POSTGRES_DB: Product
    ports:
      - "5432:5432"

  nodejs-api:
    build: .
    container_name: nodejs-api-container
    restart: always
    ports:
      - '3000:3000'
    networks:
      - node-network

networks:
  node-network:
    driver: bridge
```

Setelah memperbarui Dockerfile dan docker-compose.yml, build image Docker dan jalankan container.

```
docker-compose up --build
```

### Langkah 4: Testing API

Sekarang, API bisa di-test dengan mengakses http://172.18.46.245:3000/products di browser web Anda atau menggunakan alat seperti curl atau Postman.