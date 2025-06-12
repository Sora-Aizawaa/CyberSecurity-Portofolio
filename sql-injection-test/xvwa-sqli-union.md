🛡️ SQL Injection Practice - XVWA

🎯 Target Information
Application: XVWA (Xtreme Vulnerable Web Application)
Testing URL: http://localhost/xvwa/vulnerabilities/sqli/
Environment: Local VMware (Safe Testing Lab)
Purpose: Practicing UNION-based SQL Injection techniques in a safe, controlled environment.

🧪 Understanding Columns in UNION-Based SQL Injection
Before using a UNION SELECT statement in SQL Injection, it is important to determine the number of columns in the original SQL query. This is because the number of columns in the UNION query must match the number of columns in the original query.

🔍 Using ORDER BY to Identify the Number of Columns
To test how many columns are being used in a vulnerable SQL query, we can use the ORDER BY clause. For example:

' ORDER BY 5 #
This statement tells the database to order the results by the 5th column. If the page works without error, it means the table likely has at least 5 columns.

If you try:
' ORDER BY 6 #
and the page shows an error, it means the table does not have 6 columns — the number of columns is less than 6.

You can repeat this process to determine the exact number of columns:
' ORDER BY 1 #
' ORDER BY 2 #
' ORDER BY 3 #
Once you find the highest number that does not cause an error, that’s the number of columns in the original query.

🧪 UNION-Based SQL Injection: Extracting Data After Column Discovery
Once you have successfully identified the number of columns in the original SQL query using the ORDER BY method (for example, 7 columns), you can proceed to use the UNION SELECT statement to extract useful information from the database.

✅ Step-by-Step Example
1. Confirm the number of columns
Assume that after testing with:
' ORDER BY 7 #
you get no error, but:

' ORDER BY 8 #
produces an error. That means the SQL query has 7 columns.

2. Test with dummy data
To check which columns are reflected in the output, inject the following payload into the vulnerable input field:

' UNION SELECT 1,2,3,4,5,6,7 #
If some or all of these values appear on the page, you’ve identified which columns can be used to display data.

3. Replace with database functions
Now that you know how many columns exist and which ones are visible, you can replace specific column numbers with database functions such as:

database() — shows the current database name

user() — shows the current database user

For example:
' UNION SELECT 1, database(), 3, user(), 5, 6, 7 #
This will inject the values from the database and user functions into the page, helping you confirm that the injection is successful and data is being retrieved.

🗂️ Enumerating Tables in the Current Database
Once you have successfully injected and retrieved information using functions like database() and user(), the next step is to enumerate all the tables in the current database. This helps identify where sensitive data might be stored, such as in a users table.

🔍 Step: Extract Table Names Using information_schema
The following SQL Injection payload can be used to list all table names in the current database:

' UNION SELECT 1, GROUP_CONCAT(table_name), 3, user(), 5, 6, 7 FROM information_schema.tables WHERE table_schema = database() #

✅ Explanation:
GROUP_CONCAT(table_name): Combines all table names into a single string output.

information_schema.tables: A special MySQL system table that stores metadata about all tables.

WHERE table_schema = database(): Limits the results to the currently selected database only.

The other numbers (1, 3, user(), 5, 6, 7) match the total number of columns previously discovered (7 in this case).

📌 Result
This payload will display the list of all tables in the current database. One of the most common and interesting targets is usually a table called: users
Once you confirm that a users table exists, you can move forward and enumerate its columns and eventually the data it stores.

🔎 Enumerating Columns from the users Table
After identifying the presence of the users table, the next step is to enumerate its columns. This helps you understand what kind of data is stored — such as username, password, email, etc.

🧪 Step: Extract Column Names Using information_schema.columns
To list all column names from the users table, use the following SQL Injection payload:

' UNION SELECT 1, GROUP_CONCAT(column_name), 3, user(), 5, 6, 7 FROM information_schema.columns WHERE table_schema = database() AND table_name = 'users' #

✅ Explanation:
GROUP_CONCAT(column_name): Combines all column names into a single line.

information_schema.columns: A system table that contains metadata about columns in every table.
WHERE table_schema = database(): Filters columns to the current database.
AND table_name = 'users': Filters only columns from the users table.
The rest of the numbers (e.g., 1, 3, user(), 5, 6, 7) are placeholders to match the total of 7 columns in the query.

🧾 Example Output
The result might return something like:
id,username,password,email
These are the columns inside the users table that can be used in the next step to retrieve sensitive data.

🔐 Extracting Data from the users Table
After discovering the column names in the users table (e.g., username, password), the next objective is to extract the actual data stored within those columns — such as usernames and hashed passwords.

🧪 Step: Retrieve Data Using UNION SELECT
Use the following SQL Injection payload to extract all usernames and their corresponding passwords:

' UNION SELECT 1, GROUP_CONCAT(username, 0x3a, password), 3, user(), 5, 6, 7 FROM users #

✅ Explanation:
GROUP_CONCAT(username, 0x3a, password): Combines username and password columns with a colon (:) separator. 0x3a is the hexadecimal representation of :.

FROM users: Retrieves the data from the vulnerable users table.

Other numbers (1, 3, user(), 5, 6, 7) are used to fill the remaining columns, assuming 7 total columns.

🧾 Example Output
The page might return data like:

admin:5f4dcc3b5aa765d61d8327deb882cf99,user:test123

This means:
admin has a password hash: 5f4dcc3b5aa765d61d8327deb882cf99 (which is "password" in MD5).
user has password: test123




