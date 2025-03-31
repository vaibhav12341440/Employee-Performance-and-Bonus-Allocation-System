### Employee-Performance-and-Bonus-Allocation-System


---

### **1. Importing Required Libraries**
```python
import numpy as np
```
- The `numpy` library is imported but not used in the first few cells.

---

### **2. Reading Employee Work Hours from `timesheet.txt`**
```python
Timesheet = "timesheet.txt"
Emp_Hours = {}

with open(Timesheet, 'r') as f:
    for line in f:
        data = line.strip().split(',')
        Id = data[0]
        Hours = int(data[1])
        if Id in Emp_Hours.keys():
            Emp_Hours[Id] += Hours            
        else:
            Emp_Hours[Id] = Hours
```
- Reads data from `timesheet.txt` where each line contains:
  - `Id` → Employee ID
  - `Hours` → Hours worked
- Stores total hours worked per employee in `Emp_Hours` dictionary.

---

### **3. Processing Employee Evaluations from `evaluation.txt`**
```python
EV_Sheet = "evaluation.txt"        
EVS = {}

Pos_words = ['excellent', 'good', 'dependable', 'prompt']
Neg_words = ['poor', 'error', 'unreliable', 'late']

with open(EV_Sheet, "r") as f:
    for line in f:
        data = line.strip().split("#")
        Id = data[0]
        Remark = data[1]
        Pos = 0
        for i in Pos_words:
            Pos += Remark.count(i)
        Neg = 0
        for i in Neg_words:
            Neg += Remark.count(i)  
        if Neg == 0:
            EVS[Id] = 10.0
        else:
            EVS[Id] = round(Pos / Neg, 2)
```
- Reads `evaluation.txt`, which contains:
  - `Id` → Employee ID
  - `Remark` → Performance review comments
- Counts occurrences of positive (`Pos_words`) and negative (`Neg_words`) terms.
- Computes a score as:
  - `10.0` if there are no negative words
  - Otherwise, `score = Pos / Neg`, rounded to 2 decimals.
- Stores scores in the `EVS` dictionary.

---

### **4. Reading Sales Data from `sales.txt`**
```python
Sales = "sales.txt"
Dir_sales = {}

with open(Sales, "r") as f:
    for line in f:
        data = line.strip().split(',')
        Id, sale = data
        Dir_sales[Id] = int(sale)
```
- Reads sales performance data from `sales.txt`:
  - `Id` → Employee ID
  - `sale` → Sales amount
- Stores sales data in the `Dir_sales` dictionary.

---

### **5. Processing Employee Information from `emp_beg_yr.txt`**
```python
Emp_data = "emp_beg_yr.txt"
Directors = {}
Consultants = {}

with open(Emp_data, "r") as f:
    f.readline()  # Skip header
    for line in f:
        data = line.strip().split(',')
        Id, LN, FN, JC, BP = data

        if JC == "C":
            if Id in Emp_Hours.keys():
                PUR = round((Emp_Hours[Id] / 2250 * 100), 1)
            else:
                PUR = 0

            if Id in EVS.keys():
                ES = EVS[Id]
            else:
                ES = 1

            BP = int(BP)
            Consultants[Id] = [Id, LN, FN, BP, PUR, ES, 'C']
        
        else:
            if Id in Emp_Hours.keys():
                PUR = round((Emp_Hours[Id] / 2250 * 100), 1)
            else:
                PUR = 0

            if Id in Dir_sales.keys():
                AS = int(Dir_sales[Id])
            else:
                AS = 0

            Directors[Id] = [Id, LN, FN, BP, PUR, AS, "D"]
```
- Reads `emp_beg_yr.txt`, which contains:
  - `Id` → Employee ID
  - `LN` → Last Name
  - `FN` → First Name
  - `JC` → Job Category (`C`: Consultant, `D`: Director)
  - `BP` → Base Pay
- Computes:
  - **Consultants:**  
    - `PUR` (Percentage Utilization Rate) = `(Hours Worked / 2250) * 100`
    - `ES` (Evaluation Score) from `EVS`
  - **Directors:**  
    - `PUR` (utilization rate)
    - `AS` (Annual Sales) from `Dir_sales`
- Stores data in `Consultants` and `Directors` dictionaries.

---


---

### **6. Function: `Per_65(Dir, Con)`**
```python
def Per_65(Dir, Con):
    E_PUR = []
    for i in Dir.keys():
        E_PUR.append(Dir[i][4])
    for i in Con.keys():
        E_PUR.append(Con[i][4])
    return (np.percentile(E_PUR, 65))
```
- Computes the **65th percentile** of utilization rates (`PUR`) for all employees (Directors & Consultants).
- Uses NumPy's `percentile` function to find the utilization rate threshold for top performers.

---

### **7. Function: `Total_bonus(Dir, Con, BR)`**
```python
def Total_bonus(Dir, Con, BR):
    Total = 0
    BR = BR / 100
    PUR_65 = Per_65(Dir, Con)

    for i in Dir.keys():
        if Dir[i][4] >= PUR_65:
            if Dir[i][5] * BR <= 150000:
                Total += Dir[i][5] * BR
            else:
                Total += 150000

    for i in Con.keys():
        if Con[i][4] >= PUR_65 and Con[i][5] >= 3.5:
            if Con[i][3] * BR <= 50000:
                Total += Con[i][3] * BR
            else:
                Total += 50000

    return round(Total, 2)
```
- Computes the **total bonus payout** for all eligible employees.
- **Bonus eligibility criteria:**
  - **Directors**: Utilization rate (`PUR`) above the 65th percentile.
  - **Consultants**: `PUR` above the 65th percentile **and** evaluation score ≥ 3.5.
- **Bonus calculation:**
  - **Directors:** `Bonus = min(Sales * BR, 150,000)`
  - **Consultants:** `Bonus = min(Base Pay * BR, 50,000)`

---

### **8. Function: `Bonus(Dir, Con, Id, BR)`**
```python
def Bonus(Dir, Con, Id, BR):
    B = 0
    BR = BR / 100
    PUR_65 = Per_65(Dir, Con)

    if Id in Dir.keys():
        if Dir[Id][4] >= PUR_65:
            if Dir[Id][5] * BR <= 150000:
                B = round(Dir[Id][5] * BR, 2)
            else:
                B = 150000.00

    if Id in Con.keys():
        if Con[Id][4] >= PUR_65 and Con[Id][5] >= 3.5:
            if Con[Id][3] * BR <= 50000:
                B = round(Con[Id][3] * BR, 2)
            else:
                B = 50000.00

    return B
```
- Computes the **bonus** for a specific employee.
- Uses the same **bonus eligibility** rules as `Total_bonus()`.
- Returns the **final bonus amount** for the given employee ID.

---

### **9. Function: `Bonus_Arr(Dir, Con, BR)`**
```python
def Bonus_Arr(Dir, Con, BR):
    B_Arr = []
    BR = BR / 100
    PUR_65 = Per_65(Dir, Con)

    for Id in Dir.keys():
        if Dir[Id][4] >= PUR_65:
            if Dir[Id][5] * BR <= 150000:
                B_Arr.append(round(Dir[Id][5] * BR, 2))
            else:
                B_Arr.append(150000.00)

    for Id in Con.keys():
        if Con[Id][4] >= PUR_65 and Con[Id][5] >= 3.5:
            if Con[Id][3] * BR <= 50000:
                B_Arr.append(round(Con[Id][3] * BR, 2))
            else:
                B_Arr.append(50000.00)

    return B_Arr
```
- Creates a **list of bonuses** for all eligible employees.
- Uses the same rules as `Total_bonus()` and `Bonus()`, but stores bonuses in a list.

---

### **10. Function: `Emp_info(Dir, Con, BR, ID)`**
```python
def Emp_info(Dir, Con, BR, ID):
    if ID in Dir.keys():
        print("ID:", ID)
        print("Director:", Dir[ID][2], Dir[ID][1])
        print("Utilization:", Dir[ID][4])
        print("New sales: $" + str(Dir[ID][5]))
        print("Base pay: $" + str(Dir[ID][3]))
        print("Bonus: $" + str(Bonus(Directors, Consultants, ID, BR)))

    elif ID in Con.keys():
        print("ID:", ID)
        print("Consultant:", Con[ID][2], Con[ID][1])
        print("Utilization:", Con[ID][4])
        print("Evaluation Score:", Con[ID][5])
        print("Base pay: $" + str(Con[ID][3]))
        print("Bonus: $" + str(Bonus(Directors, Consultants, ID, BR)))
```
- Displays **detailed information** for a specific employee.
- Prints:
  - Name
  - Utilization rate
  - Sales (for Directors)
  - Evaluation score (for Consultants)
  - Base pay
  - Bonus amount

---

### **Final Code Explanation**

The last part of the notebook includes **statistical analysis, file output, and user interaction** for bonus rates and employee information.

---

### **11. Function: `stat(data)`**
```python
def stat(data):
    num_points = len(data)
    min_value = np.min(data)
    max_value = np.max(data)
    median_value = np.median(data)
    mean_value = round(np.mean(data), 2)
    std_dev = round(np.std(data), 2)

    print("Number of data points:", num_points)
    print("Minimum:", min_value)
    print("Maximum:", max_value)
    print("Median:", median_value)
    print("Mean:", mean_value)
    print("Standard deviation:", std_dev)
```
- Computes and displays **basic statistics** for a dataset:
  - Count of data points
  - Minimum and maximum values
  - Median and mean
  - Standard deviation
- Uses NumPy functions for efficient calculations.

---

### **12. Bonus Rate Input & File Output (`emp_end_yr.txt`)**
```python
while True:
    BR = float(input("Enter bonus rate: ")) 
    print("Total Bonus value for this rate is: ", Total_bonus(Directors, Consultants, BR))
    if input("Do you want to finalize this rate? (yes/no)").lower() == 'yes':
        break

valid_Id = [*Directors.keys()] + [*Consultants.keys()]
valid_id_int = [int(x) for x in valid_Id]
valid_id_int.sort()

file = open("emp_end_yr.txt", "w")
file.write("ID,LastName,FirstName,JobCode,BasePay,Utilization,Evaluation/Sales,Bonus\n")

for i in valid_id_int:
    Id = str(i)
    if Id in Directors.keys():
        emp = Directors[Id]
    else:
        emp = Consultants[Id]
    text = emp[0]+','+emp[1]+','+emp[2]+','+emp[6]+','+str(emp[3])+','+str(emp[4])+','+str(emp[5])+"\n"
    file.write(text)

file.close()
```
- **Prompts the user** to enter a bonus rate (`BR`).
- Displays the **total bonus** for the given rate.
- **Stores finalized employee data** in `"emp_end_yr.txt"`, including:
  - ID, Name, Job Code, Base Pay, Utilization, Evaluation/Sales, Bonus
- Sorts employees by ID before writing to the file.

---

### **13. Employee Lookup**
```python
while True:
    if input("\nDo you want to look up for employee info? (yes/no)").lower() == 'yes':
        ID = input("Enter employee ID: ")
        Emp_info(Directors, Consultants, BR, ID)
    else:
        break
```
- Allows the user to look up **employee details**.
- Calls `Emp_info()` to display utilization, evaluation, sales, and bonuses.
- Continues prompting until the user enters "no".

---

### **14. Statistical Summary**
```python
PUR = []
for i in Directors.keys():
    PUR.append(Directors[i][4])
for i in Consultants.keys():
    PUR.append(Consultants[i][4])

print("\nFor Utilization rate:")
stat(PUR)
print("\nFor Evaluation score:")
stat([*EVS.values()])
print("\nFor Sales:")
stat([*Dir_sales.values()])
print("\nFor Bonus:")
stat(Bonus_Arr(Directors, Consultants, BR))
```
- **Computes and prints statistics** for:
  - Utilization Rate (`PUR`)
  - Evaluation Scores (`EVS`)
  - Sales Performance (`Dir_sales`)
  - Bonuses (`Bonus_Arr`)

---

### **15. Identifying Top Performers**
```python
Max_PUR = np.max(PUR)
Max_Sales = np.max([*Dir_sales.values()])

for i in Directors.keys():
    if Directors[i][4] == Max_PUR:
        print("\nDirector with highest utilization rate: ")
        Emp_info(Directors, Consultants, BR, i)

for i in Consultants.keys():
    if Consultants[i][4] == Max_PUR:
        print("\nConsultant with highest utilization rate: ")
        Emp_info(Directors, Consultants, BR, i)

for i in Directors.keys():
    if Directors[i][5] == Max_Sales:
        print("\nDirector with highest sales: ")
        Emp_info(Directors, Consultants, BR, i)
```
- Identifies:
  - **Director & Consultant with the highest Utilization Rate**
  - **Director with the highest Sales Performance**
- Calls `Emp_info()` to display details.


---

### **16. Creating an Error Log File**
```python
error_log = open('error_log.txt', 'w')
```
- Opens (or creates) a file named `error_log.txt` in **write mode (`'w'`)**.
- Any detected errors will be written to this file.

---

### **17. Checking the `timesheet.txt` File for Errors**
```python
f = open(Timesheet, 'r') 
error_log.write("Error log of Timesheet file:\n")

for line in f:
    data = line.strip().split(',')
    if data[0] not in valid_Id:
        error_log.write(data[0] + '\n')
f.close()
```
- Reads `timesheet.txt` line by line.
- Extracts the **employee ID** (`data[0]`).
- If the ID is **not in `valid_Id`**, it is **logged as an error**.
- `valid_Id` contains all valid employee IDs from `emp_beg_yr.txt`.
- Writes **invalid IDs** to `error_log.txt`.

---

### **18. Checking the `evaluation.txt` File for Errors**
```python
f = open(EV_Sheet, 'r') 
error_log.write("\nError log of Evaluation file:\n")

for line in f:
    data = line.strip().split('#')
    if data[0] not in valid_Id:
        error_log.write(data[0] + "\n")
f.close()
```
- Reads `evaluation.txt` and extracts employee IDs.
- If an employee ID is **not found in `valid_Id`**, it is logged.
- The delimiter is `#` (instead of `,` as in `timesheet.txt`).
- Writes errors under `"Error log of Evaluation file"` in `error_log.txt`.

---

### **19. Checking the `sales.txt` File for Errors**
```python
f = open(Sales, 'r')
error_log.write("\nError log of Sales file:\n")

for line in f:
    data = line.strip().split(',')
    if data[0] not in valid_Id:
        error_log.write(data[0] + '\n')
f.close()
```
- Reads `sales.txt`, extracts employee IDs.
- Checks if each ID exists in `valid_Id`.
- Logs any **invalid employee IDs**.

---

### **20. Closing the Error Log File**
```python
error_log.close()
```
- Closes the error log file to save changes.

---

### **Purpose of This Code**
✔ Ensures **data integrity** by identifying **invalid employee IDs**.  
✔ Helps **debug issues** in `timesheet.txt`, `evaluation.txt`, and `sales.txt`.  
✔ Generates `error_log.txt` to track missing or incorrect employee records.  

---

### **Example of `error_log.txt` Output**
```
Error log of Timesheet file:
9999
8888

Error log of Evaluation file:
7777

Error log of Sales file:
6666
```
This means:
- Employee ID `9999` and `8888` exist in `timesheet.txt` but not in `emp_beg_yr.txt`.
- Employee ID `7777` appears in `evaluation.txt` but is not valid.
- Employee ID `6666` is listed in `sales.txt` but doesn't exist in employee records.

---


