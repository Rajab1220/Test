Steps to Run Your Library Management System in VS Code
Follow these steps to set up and run your project properly.
________________________________________
1️⃣ Setup the Virtual Environment (If Not Already Done)
Open VS Code Terminal and navigate to your project folder:
        cd path/to/your/project
Then, create and activate a virtual environment.
On Windows (Command Prompt or PowerShell)
        python -m venv venv
        venv\Scripts\activate
________________________________________
2️⃣ Install Required Dependencies
Run the following command to install the necessary packages:
        pip install fastapi uvicorn sqlalchemy passlib[bcrypt] requests
________________________________________
3️⃣ Initialize the Database
Run the following command to create the SQLite database and tables:
        python init_db.py
✅ This will generate library.db with all required tables.
________________________________________
4️⃣ Start the FastAPI Server
Run the following command to start the FastAPI server:
        uvicorn main:app --reload
✅ This will start the API server on http://127.0.0.1:8000.
________________________________________
5️⃣ Run the CLI Application
Open another terminal (don’t close the FastAPI server terminal) and run:
        python cli.py
✅ This will launch the interactive command-line interface (CLI) for interacting with the system.
________________________________________
6️⃣ Testing API with Swagger UI
You can also test API endpoints using the built-in Swagger UI at: 🔗 http://127.0.0.1:8000/docs
This provides an easy way to test the API without writing extra code.
________________________________________
7️⃣ Stopping the Application
•	Stop CLI: Press Ctrl + C in the terminal.
•	Stop FastAPI Server: Press Ctrl + C in the terminal where FastAPI is running.
________________________________________
🎯 Final Execution Order
1.	Initialize the database: python init_db.py
2.	Start the FastAPI server: uvicorn main:app --reload
3.	Run the CLI interface: python cli.py

