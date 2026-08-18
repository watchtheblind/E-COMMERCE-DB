# 🛒 E-Commerce Database Project

This project provides a relational database architecture for a high-concurrency E-Commerce system, optimized for PostgreSQL and prepared for advanced implementations like caching (Redis), indexing, and ACID transactions.

---

## 🚀 Docker Deployment

To launch the database container, use the following `docker-compose.yml` configuration:

```yaml
version: '3.8'
services:
  postgres_db:
    image: postgres:16
    container_name: ecommerce_postgres
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: ecommerce
    ports:
      - "5432:5432"
    volumes:
      - /media/luismonasterios/Nuevo vol/var/lib/postgresql/data:/var/lib/postgresql/data

```

### 💾 Storing Data on a Secondary Disk

To ensure that massive database records do not fill up your primary operating system disk, the volume configuration maps directly to the external partition (`/dev/sdc1`) mounted at `/media/luismonasterios/Nuevo vol/`.

1. Ensure your system has read and write permissions configured for the target directory.
2. Run the container in the background using the terminal command:
```bash
docker-compose up -d

```



---

## 🛠️ Connection Parameters for DBeaver

Once the container is running successfully, connect using these credentials:

* **Host:** `localhost`
* **Puerto:** `5432`
* **Database:** `ecommerce`
* **User:** `admin`
* **Password:** `mysecretpassword`

---

## 🏗️ Database Schema

The complete PostgreSQL script featuring `UUID` primary keys, foreign key relationships, and `CHECK` constraints for data integrity:

```sql
-- 1. Independent Tables
CREATE TABLE CATEGORIES (
    ID SERIAL PRIMARY KEY,
    NAME VARCHAR(255) NOT NULL
);

CREATE TABLE COUNTRIES (
    ID SERIAL PRIMARY KEY,
    COUNTRY_NAME VARCHAR(150) NOT NULL,
    ISO_CODE CHAR(3) NOT NULL
);

-- 2. Users and Details (1:1 Relationship)
CREATE TABLE USERS (
    ID UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    USERNAME VARCHAR(50) NOT NULL UNIQUE,
    PASSWORD VARCHAR(50) NOT NULL,
    PHONE_NUMBER VARCHAR(20) NOT NULL UNIQUE,
    EMAIL VARCHAR(255) NOT NULL UNIQUE,
    ISVERIFIED BOOLEAN NOT NULL DEFAULT FALSE,
    USER_DETAILS_ID UUID,
    CONSTRAINT VALID_EMAIL CHECK (EMAIL ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT VALID_PHONE CHECK (PHONE_NUMBER ~* '^\+?[1-9]\d{1,14}$')
);

CREATE TABLE USER_DETAILS (
    ID UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    FIRST_NAME VARCHAR(50),
    LAST_NAME VARCHAR(50),
    COUNTRY_ID INT,
    USER_ID UUID,
    DIRECTION_1 VARCHAR(150),
    DIRECTION_2 VARCHAR(150),
    FOREIGN KEY (COUNTRY_ID) REFERENCES COUNTRIES(ID),
    FOREIGN KEY (USER_ID) REFERENCES USERS(ID)
);

-- 3. Products
CREATE TABLE PRODUCTS (
    ID UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    NAME VARCHAR(100) NOT NULL,
    CATEGORY_ID INT,
    STOCK INT CHECK (STOCK >= 0),
    PRICE DECIMAL(10,2) CHECK (PRICE > 0.00),
    FOREIGN KEY (CATEGORY_ID) REFERENCES CATEGORIES(ID)
);

-- 4. Orders and Order Items
CREATE TABLE ORDERS (
    ID UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    USER_ID UUID,
    CREATED_AT TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    TOTAL_AMOUNT DECIMAL(10,2),
    STATUS VARCHAR(25),
    CONSTRAINT CHK_STATUS CHECK (STATUS IN ('PENDING', 'PAID', 'SHIPPED', 'CANCELLED')),
    FOREIGN KEY (USER_ID) REFERENCES USERS(ID)
);

CREATE TABLE ORDER_ITEMS (
    ID UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ORDER_ID UUID,
    PRODUCT_ID UUID,
    QUANTITY INT NOT NULL,
    UNIT_PRICE DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (ORDER_ID) REFERENCES ORDERS(ID),
    FOREIGN KEY (PRODUCT_ID) REFERENCES PRODUCTS(ID)
);

```