Here's a complete implementation of an Inventory Management System using FastAPI and MySQL instead of Flask. This system includes database setup, API endpoints, and a command-line interface.


---

1. Database Setup (MySQL)

Create a database and tables using the following SQL script:

CREATE DATABASE inventory_db;

USE inventory_db;

CREATE TABLE Products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    category VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    quantity INT NOT NULL
);

CREATE TABLE Transactions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    type ENUM('purchase', 'sale') NOT NULL,
    quantity INT NOT NULL,
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES Products(id) ON DELETE CASCADE
);


---

2. Python Backend using FastAPI

Install Dependencies

Run the following command to install required packages:

pip install fastapi uvicorn mysql-connector-python pydantic sqlalchemy


---

Main API File (main.py)

from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel
from sqlalchemy import create_engine, Column, Integer, String, DECIMAL, Enum, ForeignKey, TIMESTAMP
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
import datetime

DATABASE_URL = "mysql+mysqlconnector://root:password@localhost/inventory_db"

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine, autoflush=False)
Base = declarative_base()

app = FastAPI()

# Database Models
class Product(Base):
    __tablename__ = "Products"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(255), nullable=False)
    category = Column(String(100), nullable=False)
    price = Column(DECIMAL(10,2), nullable=False)
    quantity = Column(Integer, nullable=False)

class Transaction(Base):
    __tablename__ = "Transactions"
    id = Column(Integer, primary_key=True, index=True)
    product_id = Column(Integer, ForeignKey("Products.id", ondelete="CASCADE"), nullable=False)
    type = Column(Enum("purchase", "sale"), nullable=False)
    quantity = Column(Integer, nullable=False)
    transaction_date = Column(TIMESTAMP, default=datetime.datetime.utcnow)

# Pydantic Schemas
class ProductCreate(BaseModel):
    name: str
    category: str
    price: float
    quantity: int

class ProductUpdate(BaseModel):
    name: str
    category: str
    price: float
    quantity: int

class TransactionCreate(BaseModel):
    product_id: int
    type: str
    quantity: int

# Dependency for DB Session
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# API Endpoints
@app.post("/products/")
def add_product(product: ProductCreate, db: Session = Depends(get_db)):
    new_product = Product(**product.dict())
    db.add(new_product)
    db.commit()
    db.refresh(new_product)
    return new_product

@app.get("/products/")
def list_products(db: Session = Depends(get_db)):
    return db.query(Product).all()

@app.put("/products/{product_id}")
def update_product(product_id: int, product: ProductUpdate, db: Session = Depends(get_db)):
    db_product = db.query(Product).filter(Product.id == product_id).first()
    if not db_product:
        raise HTTPException(status_code=404, detail="Product not found")
    for key, value in product.dict().items():
        setattr(db_product, key, value)
    db.commit()
    db.refresh(db_product)
    return db_product

@app.post("/transactions/")
def record_transaction(transaction: TransactionCreate, db: Session = Depends(get_db)):
    product = db.query(Product).filter(Product.id == transaction.product_id).first()
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    
    if transaction.type == "sale" and product.quantity < transaction.quantity:
        raise HTTPException(status_code=400, detail="Insufficient stock")

    if transaction.type == "sale":
        product.quantity -= transaction.quantity
    else:
        product.quantity += transaction.quantity

    db.add(Transaction(**transaction.dict()))
    db.commit()
    return {"message": "Transaction recorded successfully"}

@app.get("/transactions/{product_id}")
def view_transactions(product_id: int, db: Session = Depends(get_db)):
    transactions = db.query(Transaction).filter(Transaction.product_id == product_id).all()
    if not transactions:
        raise HTTPException(status_code=404, detail="No transactions found")
    return transactions


---

3. Command-Line Interface (cli.py)

import requests

BASE_URL = "http://127.0.0.1:8000"

def add_product():
    name = input("Enter product name: ")
    category = input("Enter category: ")
    price = float(input("Enter price: "))
    quantity = int(input("Enter quantity: "))
    
    response = requests.post(f"{BASE_URL}/products/", json={"name": name, "category": category, "price": price, "quantity": quantity})
    print(response.json())

def list_products():
    response = requests.get(f"{BASE_URL}/products/")
    print(response.json())

def update_product():
    product_id = int(input("Enter product ID to update: "))
    name = input("Enter new name: ")
    category = input("Enter new category: ")
    price = float(input("Enter new price: "))
    quantity = int(input("Enter new quantity: "))
    
    response = requests.put(f"{BASE_URL}/products/{product_id}", json={"name": name, "category": category, "price": price, "quantity": quantity})
    print(response.json())

def record_transaction():
    product_id = int(input("Enter product ID: "))
    transaction_type = input("Enter transaction type (purchase/sale): ")
    quantity = int(input("Enter quantity: "))

    response = requests.post(f"{BASE_URL}/transactions/", json={"product_id": product_id, "type": transaction_type, "quantity": quantity})
    print(response.json())

def view_transactions():
    product_id = int(input("Enter product ID to view transactions: "))
    response = requests.get(f"{BASE_URL}/transactions/{product_id}")
    print(response.json())

def menu():
    while True:
        print("\nInventory Management System")
        print("1. Add Product")
        print("2. List Products")
        print("3. Update Product")
        print("4. Record Transaction")
        print("5. View Transactions")
        print("6. Exit")

        choice = input("Enter choice: ")
        if choice == "1":
            add_product()
        elif choice == "2":
            list_products()
        elif choice == "3":
            update_product()
        elif choice == "4":
            record_transaction()
        elif choice == "5":
            view_transactions()
        elif choice == "6":
            break
        else:
            print("Invalid choice, try again.")

if __name__ == "__main__":
    menu()


---

4. Running the Application

1. Start MySQL Server and ensure inventory_db exists.


2. Run FastAPI Server:

uvicorn main:app --reload


3. Run the CLI:

python cli.py




---

This implementation: ✔ Uses FastAPI for performance.
✔ Uses MySQL as the database.
✔ Implements CRUD operations for products.
✔ Implements transaction management for inventory.
✔ Provides a CLI for user interaction.

Let me know if you need enhancements!

