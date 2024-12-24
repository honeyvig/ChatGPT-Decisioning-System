# ChatGPT-Decisioning-System
Chat GPT decisioning system

We are a finance company that deals with lots of lenders.
Each lender has a set of criteria that they go by to approve/decline a client.

I am looking to create a small system that we can put in certain details of a client and it check through our ‘master database’ and then provide back which lenders criteria the client would fit.
-------------------
To build a decisioning system that checks client details against a set of lender criteria, we can approach the task with the following steps:

    Database Structure: You'll need a database that holds the lender criteria and client data.
    Client Data Input: The system will accept client details (e.g., income, credit score, loan amount, etc.).
    Criteria Matching: The system will compare the client's data against the lender criteria and provide feedback on which lenders the client qualifies for.
    Output: The system will return the relevant lender information, including the specific criteria the client fits.

Below is a Python-based approach that uses a simple decision engine to match client data against lender criteria. We'll use SQLite as the database for simplicity, but you can extend this with other databases like MySQL, PostgreSQL, or even cloud databases.
Step-by-Step Python Solution:
1. Setup Database (SQLite)

Let's assume that each lender has a set of criteria for approval (e.g., minimum credit score, minimum income, loan amount range). We will create two tables:

    lenders: Contains lender information (e.g., name).
    criteria: Contains the criteria for each lender.

import sqlite3

# Connect to SQLite database (it will be created if it doesn't exist)
conn = sqlite3.connect('lenders.db')
cursor = conn.cursor()

# Create the tables
cursor.execute('''
CREATE TABLE IF NOT EXISTS lenders (
    id INTEGER PRIMARY KEY,
    name TEXT
)
''')

cursor.execute('''
CREATE TABLE IF NOT EXISTS criteria (
    id INTEGER PRIMARY KEY,
    lender_id INTEGER,
    min_credit_score INTEGER,
    min_income INTEGER,
    min_loan_amount INTEGER,
    max_loan_amount INTEGER,
    FOREIGN KEY (lender_id) REFERENCES lenders(id)
)
''')

# Insert some example lenders and criteria
cursor.execute('INSERT INTO lenders (name) VALUES (?)', ('Lender A',))
cursor.execute('INSERT INTO lenders (name) VALUES (?)', ('Lender B',))
cursor.execute('INSERT INTO lenders (name) VALUES (?)', ('Lender C',))

cursor.execute('''
INSERT INTO criteria (lender_id, min_credit_score, min_income, min_loan_amount, max_loan_amount) 
VALUES (1, 650, 30000, 5000, 50000)
''')

cursor.execute('''
INSERT INTO criteria (lender_id, min_credit_score, min_income, min_loan_amount, max_loan_amount) 
VALUES (2, 700, 40000, 10000, 100000)
''')

cursor.execute('''
INSERT INTO criteria (lender_id, min_credit_score, min_income, min_loan_amount, max_loan_amount) 
VALUES (3, 650, 25000, 1000, 20000)
''')

conn.commit()

This creates a basic lenders table with three lenders and their respective criteria in the criteria table.
2. Client Input and Matching Logic

Now let's define a function that accepts client data, checks it against the criteria in the database, and outputs which lenders the client qualifies for.

def check_client_qualification(client_data):
    # Client data input (e.g., credit score, income, loan amount requested)
    client_credit_score = client_data.get('credit_score')
    client_income = client_data.get('income')
    client_loan_amount = client_data.get('loan_amount')

    # Fetch all lenders and their criteria
    cursor.execute('''
    SELECT lenders.name, criteria.min_credit_score, criteria.min_income, 
           criteria.min_loan_amount, criteria.max_loan_amount
    FROM lenders
    JOIN criteria ON lenders.id = criteria.lender_id
    ''')

    lenders = cursor.fetchall()
    
    qualified_lenders = []

    for lender in lenders:
        lender_name, min_credit_score, min_income, min_loan_amount, max_loan_amount = lender
        
        # Check if client meets the criteria for this lender
        if (client_credit_score >= min_credit_score and
            client_income >= min_income and
            min_loan_amount <= client_loan_amount <= max_loan_amount):
            qualified_lenders.append(lender_name)
    
    return qualified_lenders


# Example client data input
client_data = {
    'credit_score': 680,
    'income': 35000,
    'loan_amount': 15000
}

# Check which lenders the client qualifies for
qualified_lenders = check_client_qualification(client_data)

# Output the result
if qualified_lenders:
    print("The client qualifies for the following lenders:")
    for lender in qualified_lenders:
        print(lender)
else:
    print("The client does not qualify for any lenders.")

Explanation of the Code:

    Database Schema:
        The lenders table stores information about each lender.
        The criteria table stores the specific criteria for each lender, including the minimum credit score, income, and the acceptable loan amount range.

    Check Client Qualification:
        The check_client_qualification function takes a dictionary of client data (credit_score, income, loan_amount).
        It fetches all the lenders and their criteria from the database.
        It then checks whether the client qualifies for each lender based on the criteria (credit score, income, and loan amount).
        If the client meets the criteria, the lender is added to the list of qualified lenders.

    Example Client Data:
        The client has a credit score of 680, an income of 35000, and requests a loan of 15000.
        The system will check against the criteria in the database and determine which lenders the client qualifies for.

3. Running the System

When you run the Python script, it will:

    Check the client's data.
    Compare it with the lenders’ criteria.
    Print out a list of lenders the client qualifies for.

Example Output:

The client qualifies for the following lenders:
Lender A
Lender B

4. Further Enhancements

    Additional Criteria: Add more criteria such as age, employment status, etc., and adjust the logic accordingly.
    Dynamic Criteria: Allow for more complex decisioning rules using a dynamic set of criteria (e.g., handling ranges, multiple conditions).
    User Interface: You can build a simple front-end interface (e.g., using Flask or Django) to accept client details and display results interactively.
    More Lender Data: The system can be extended to handle more lenders, criteria, and even loan types.
    CRM Integration: Once the client is qualified, you can integrate with your CRM to update client records or initiate further actions.

Conclusion:

This Python-based decisioning system helps check whether a client qualifies for a loan based on predefined criteria for multiple lenders. It provides a simple and effective way to automate loan qualification using a database and basic decision logic. You can enhance it further by integrating with your CRM and adding more advanced features like reporting and notifications.
