# pro1_personal-expense-tracker
"""
Personal Expense Tracker
A console-based application to record, manage, search, and analyze daily expenses.

Author: (Your Name Here)
"""

import json
import os
from datetime import datetime

# File used to permanently store expense data
DATA_FILE = "expenses.json"


# ---------------------------------------------------------------------------
# File Handling Functions (Step 9)
# ---------------------------------------------------------------------------

def load_expenses():
    """Load expenses from the JSON file when the application starts.
    Returns an empty list if the file doesn't exist or is corrupted.
    """
    if not os.path.exists(DATA_FILE):
        return []

    try:
        with open(DATA_FILE, "r") as file:
            data = json.load(file)
            if isinstance(data, list):
                return data
            return []
    except (json.JSONDecodeError, IOError):
        print("Warning: Could not read existing data file. Starting fresh.")
        return []


def save_expenses(expenses):
    """Save the current list of expenses to the JSON file."""
    try:
        with open(DATA_FILE, "w") as file:
            json.dump(expenses, file, indent=4)
    except IOError:
        print("Error: Could not save expenses to file.")


# ---------------------------------------------------------------------------
# Validation Helpers (Step 8)
# ---------------------------------------------------------------------------

def is_valid_date(date_text):
    """Validate that the date is in DD-MM-YYYY format."""
    try:
        datetime.strptime(date_text, "%d-%m-%Y")
        return True
    except ValueError:
        return False


def get_valid_date():
    """Prompt the user until a valid date is entered."""
    while True:
        date_input = input("Enter Date (DD-MM-YYYY): ").strip()
        if is_valid_date(date_input):
            return date_input
        print("Invalid Input. Please Try Again.")


def get_valid_category():
    """Prompt the user until a non-empty category is entered."""
    while True:
        category = input("Enter Category: ").strip()
        if category:
            return category
        print("Invalid Input. Please Try Again.")


def get_valid_amount():
    """Prompt the user until a valid, non-negative amount is entered."""
    while True:
        amount_input = input("Enter Amount: ").strip()
        try:
            amount = float(amount_input)
            if amount < 0:
                print("Invalid Input. Please Try Again.")
                continue
            return amount
        except ValueError:
            print("Invalid Input. Please Try Again.")


def get_valid_description():
    """Prompt the user until a non-empty description is entered."""
    while True:
        description = input("Enter Description: ").strip()
        if description:
            return description
        print("Invalid Input. Please Try Again.")


def format_amount(amount):
    """Format a number as currency for display, e.g. 5250 -> 5,250"""
    if amount == int(amount):
        return f"{int(amount):,}"
    return f"{amount:,.2f}"


# ---------------------------------------------------------------------------
# Step 2: Add Expense
# ---------------------------------------------------------------------------

def add_expense(expenses):
    """Collect expense details from the user and add them to the list."""
    print("\n--- Add Expense ---")
    date = get_valid_date()
    category = get_valid_category()
    amount = get_valid_amount()
    description = get_valid_description()

    expense = {
        "date": date,
        "category": category,
        "amount": amount,
        "description": description
    }

    expenses.append(expense)
    save_expenses(expenses)
    print("\nExpense Added Successfully.")


# ---------------------------------------------------------------------------
# Step 3: View Expenses
# ---------------------------------------------------------------------------

def view_expenses(expenses):
    """Display all expense records."""
    print("\n--- View Expenses ---")
    if not expenses:
        print("No Expense Records Found.")
        return

    for index, expense in enumerate(expenses, start=1):
        print(f"\nRecord {index}")
        print(f"Date        : {expense['date']}")
        print(f"Category    : {expense['category']}")
        print(f"Amount      : {format_amount(expense['amount'])}")
        print(f"Description : {expense['description']}")


# ---------------------------------------------------------------------------
# Step 4: Search Expense
# ---------------------------------------------------------------------------

def search_expense(expenses):
    """Search for expenses by category (case-insensitive)."""
    print("\n--- Search Expense ---")
    if not expenses:
        print("No Expense Records Found.")
        return

    category = input("Enter Category: ").strip()
    if not category:
        print("Invalid Input. Please Try Again.")
        return

    matches = [e for e in expenses if e["category"].lower() == category.lower()]

    if not matches:
        print("Expense Not Found.")
        return

    for index, expense in enumerate(matches, start=1):
        print(f"\nRecord {index}")
        print(f"Date        : {expense['date']}")
        print(f"Category    : {expense['category']}")
        print(f"Amount      : {format_amount(expense['amount'])}")
        print(f"Description : {expense['description']}")


# ---------------------------------------------------------------------------
# Step 5: Delete Expense
# ---------------------------------------------------------------------------

def delete_expense(expenses):
    """Delete an expense using Category or Description."""
    print("\n--- Delete Expense ---")
    if not expenses:
        print("No Expense Records Found.")
        return

    print("Delete by:")
    print("1. Category")
    print("2. Description")
    choice = input("Enter Choice: ").strip()

    if choice == "1":
        key = "category"
        value = input("Enter Category: ").strip()
    elif choice == "2":
        key = "description"
        value = input("Enter Description: ").strip()
    else:
        print("Invalid Input. Please Try Again.")
        return

    if not value:
        print("Invalid Input. Please Try Again.")
        return

    for expense in expenses:
        if expense[key].lower() == value.lower():
            expenses.remove(expense)
            save_expenses(expenses)
            print("\nExpense Deleted Successfully.")
            return

    print("Expense Not Found.")


# ---------------------------------------------------------------------------
# Step 6: Calculate Total Spending
# ---------------------------------------------------------------------------

def calculate_total_spending(expenses):
    """Calculate and display the total of all recorded expenses."""
    print("\n--- Total Spending ---")
    if not expenses:
        print("No Expense Records Found.")
        return

    total = sum(expense["amount"] for expense in expenses)
    print(f"Total Spending: \u20b9{format_amount(total)}")


# ---------------------------------------------------------------------------
# Step 10: Generate Expense Summary
# ---------------------------------------------------------------------------

def expense_summary(expenses):
    """Display a summary of all expenses: totals, highest expense, top category."""
    print("\n--- Expense Summary ---")
    if not expenses:
        print("No Expense Records Found.")
        return

    total_records = len(expenses)
    total_spending = sum(expense["amount"] for expense in expenses)
    highest_expense = max(expenses, key=lambda e: e["amount"])

    category_totals = {}
    for expense in expenses:
        category = expense["category"]
        category_totals[category] = category_totals.get(category, 0) + expense["amount"]
    top_category = max(category_totals, key=category_totals.get)

    print(f"Total Records  : {total_records}")
    print(f"Total Spending : \u20b9{format_amount(total_spending)}")
    print(f"Highest Expense: \u20b9{format_amount(highest_expense['amount'])}")
    print(f"Top Category   : {top_category}")


# ---------------------------------------------------------------------------
# Step 1: Main Menu
# ---------------------------------------------------------------------------

def display_menu():
    print("\n===== Personal Expense Tracker =====")
    print("1. Add Expense")
    print("2. View Expenses")
    print("3. Search Expense")
    print("4. Delete Expense")
    print("5. Calculate Total Spending")
    print("6. Expense Summary")
    print("7. Exit")


def main():
    expenses = load_expenses()

    while True:
        display_menu()
        choice = input("Enter Choice: ").strip()

        if choice == "1":
            add_expense(expenses)
        elif choice == "2":
            view_expenses(expenses)
        elif choice == "3":
            search_expense(expenses)
        elif choice == "4":
            delete_expense(expenses)
        elif choice == "5":
            calculate_total_spending(expenses)
        elif choice == "6":
            expense_summary(expenses)
        elif choice == "7":
            save_expenses(expenses)
            print("\nThank you for using Personal Expense Tracker. Goodbye!")
            break;
        else:
            print("Invalid Input. Please Try Again.")


if __name__ == "__main__":
    main()
