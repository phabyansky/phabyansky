import datetime
from tkinter import *
import tkinter.messagebox as mb
from tkinter import ttk
from tkcalendar import DateEntry
import sqlite3
from tkinter import filedialog
from tkinter import ttk, Scrollbar
from tkinter import HORIZONTAL, VERTICAL, RIGHT, BOTTOM, CENTER
import mysql.connector

headlabelfont = ("Noto Sans CJK TC", 15, 'bold')
labelfont = ('Garamond', 12)
entryfont = ('Garamond', 12)

connector = mysql.connector.connect(
    host="localhost",
    user="root",
    password="baby1",
    database="SchoolManagement"
)

cursor = connector.cursor()  # Create a cursor object from the connection

# Now you can use the cursor to execute queries
cursor.execute("SELECT * FROM SCHOOL_MANAGEMENT")  # Use 'SCHOOL_MANAGEMENT' in the query
data = cursor.fetchall()

# Process the fetched data or perform other operations

cursor.close()  # Close the cursor when finished
connector.close()  # Close the connection

connector = mysql.connector.connect(
    host="localhost",
    user="root",
    password="baby1",
    database="SchoolManagement"  # Replace with your database name
)

# Create a cursor object from the connection
cursor = connector.cursor()

# Define the SQL query to create the 'SCHOOL_MANAGEMENT' table
create_school_management_table = """
CREATE TABLE IF NOT EXISTS SCHOOL_MANAGEMENT (
    STUDENT_ID INT AUTO_INCREMENT PRIMARY KEY,
    NAME VARCHAR(100),
    EMAIL VARCHAR(100),
    PHONE_NO VARCHAR(20),
    GENDER VARCHAR(10),
    DOB DATE,
    STREAM VARCHAR(50),
    GRADE VARCHAR(10),
    REGISTRATION_NUMBER VARCHAR(20),
    SCHOOL VARCHAR(100),
    DEPARTMENT VARCHAR(100),
    ADDRESS VARCHAR(255)
)
"""

# Execute the table creation query
try:
    cursor.execute(create_school_management_table)
    print("SCHOOL_MANAGEMENT table created successfully (or already exists).")
except mysql.connector.Error as err:
    print(f"Error creating SCHOOL_MANAGEMENT table: {err}")

def reset_fields():
    global name_strvar, email_strvar, contact_strvar, gender_strvar, dob, stream_strvar, grade_strvar,registration_number_strvar,school_strvar,department_strvar, address_strvar
    name_strvar.set('')
    email_strvar.set('')
    contact_strvar.set('')
    gender_strvar.set('')
    stream_strvar.set('')
    grade_strvar.set('')
    dob.set_date(datetime.datetime.now().date())
    registration_number_strvar.set('')
    school_strvar.set('')
    department_strvar.set("")
    address_strvar.set('')

# Define a global variable to track the toggle state
toggle_state = True  # Set initial state to True (for displaying records)

def toggle_clear_records():
    global toggle_state
    if toggle_state:
        tree.delete(*tree.get_children())  # Clear the window
        reset_fields()  # Reset form fields
        toggle_state = False  # Update toggle state
        toggle_button.config(text='Refresh')  # Change button text to 'Refresh'
    else:
        display_records()  # Display records
        toggle_state = True  # Update toggle state
        toggle_button.config(text='Clear Window')  # Change button text to 'Clear Window'

def display_records():
    tree.delete(*tree.get_children())

    # Fetch all records from the database
    cursor.execute('SELECT * FROM SCHOOL_MANAGEMENT')
    data = cursor.fetchall()

    # Display the fetched records in the Treeview with updated Serial/No column
    serial_number = 1  # Initialize serial number counter
    for record in data:
        # Insert records with updated serial numbers
        tree.insert('', END, values=(serial_number, *record[1:]))
        serial_number += 1  # Increment serial number for the next record

def add_record():
    name = name_strvar.get()
    email = email_strvar.get()
    contact = contact_strvar.get()
    gender = gender_strvar.get()
    DOB = dob.get()
    stream = stream_strvar.get()
    grade = grade_strvar.get()
    registration_number = registration_number_strvar.get()
    school = school_strvar.get()
    department = department_strvar.get()
    address = address_strvar.get()

    # Reformat the date string from 'MM/DD/YYYY' to 'YYYY-MM-DD'
    parts = DOB.split('/')
    formatted_date = f'{parts[2]}-{parts[0].zfill(2)}-{parts[1].zfill(2)}'

    if not name or not email or not contact or not gender or not DOB or not stream or not address or not grade or not registration_number or not school or not department:
        mb.showerror('Error!', "Please fill all the missing fields!!")
    else:
        try:
            cursor.execute(
                'INSERT INTO SCHOOL_MANAGEMENT (NAME, EMAIL, PHONE_NO, GENDER, DOB, STREAM, GRADE, REGISTRATION_NUMBER, SCHOOL, DEPARTMENT, ADDRESS) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)',
                (name, email, contact, gender, formatted_date, stream, grade, registration_number, school, department,
                 address)
            )
            connector.commit()
            mb.showinfo('Record added', f"Record of {name} was successfully added")
            reset_fields()
            display_records()  # Refresh displayed records after adding
        except mysql.connector.Error as e:
            mb.showerror('Error', f'Failed to add record: {e}')

def remove_record():
    if not tree.selection():
        mb.showerror('Error!', 'Please select an item from the database')
    else:
        try:
            current_item = tree.focus()
            values = tree.item(current_item)
            selection = values["values"]
            tree.delete(current_item)
            connector.execute('DELETE FROM SCHOOL_MANAGEMENT WHERE STUDENT_ID=?', (selection[0],))
            connector.commit()
            mb.showinfo('Done', 'The record you wanted deleted was successfully deleted.')
            display_records()
        except sqlite3.Error as e:
            mb.showerror('Error', f'Failed to delete record: {e}')

def view_record():
    selected_item = tree.selection()
    if not selected_item:
        mb.showerror('Error!', 'Please select a record to view')
    else:
        item_values = tree.item(selected_item)
        selection = item_values["values"]

        name_strvar.set(selection[1])  # Assuming Name is at index 0
        email_strvar.set(selection[2])  # Assuming Email is at index 1
        contact_strvar.set(selection[3])  # Assuming Contact is at index 2
        gender_strvar.set(selection[4])  # Assuming Gender is at index 3

        # Rearrange the date format 'YYYY-MM-DD' to 'MM/DD/YYYY'
        date_parts = selection[5].split('-')  # Assuming Date of Birth is at index 4
        dob_date = f"{date_parts[1]}/{date_parts[2]}/{date_parts[0]}"
        dob.set_date(dob_date)

        stream_strvar.set(selection[6])  # Assuming Stream is at index 5
        grade_strvar.set(selection[7])  # Assuming Grade is at index 6
        registration_number_strvar.set(selection[8])  # Assuming Registration No is at index 7
        school_strvar.set(selection[9])  # Assuming School is at index 8
        department_strvar.set(selection[10])  # Assuming Department is at index 9
        address_strvar.set(selection[11])  # Assuming Address is at index 10
def edit_record():
    selected_item = tree.selection()
    if not selected_item:
        mb.showerror('Error!', 'Please select a record to edit')
    else:
        item_values = tree.item(selected_item)
        selection = item_values["values"]

        name = name_strvar.get()
        email = email_strvar.get()
        contact = contact_strvar.get()
        gender = gender_strvar.get()
        DOB = dob.get()
        stream = stream_strvar.get()
        grade = grade_strvar.get()
        registration_number = registration_number_strvar.get()
        school = school_strvar.get()
        department = department_strvar.get()
        address = address_strvar.get()

        if not name or not email or not contact or not gender or not DOB or not stream or not address or not grade or not registration_number or not school or not department:
            mb.showerror('Error!', "Please fill all the missing fields!!")
        else:
            try:
                # Update the record in the database
                cursor.execute('UPDATE SCHOOL_MANAGEMENT SET NAME=?, EMAIL=?, PHONE_NO=?, GENDER=?, DOB=?, STREAM=?, GRADE=?, REGISTRATION_NUMBER=?, SCHOOL=?, DEPARTMENT=?, ADDRESS=? WHERE STUDENT_ID=?',
                               (name, email, contact, gender, DOB, stream, grade, registration_number, school, department, address, selection[0]))
                connector.commit()
                mb.showinfo('Record updated', f"Record with ID {selection[0]} was successfully updated")
                reset_fields()  # Reset fields after update
                display_records()  # Refresh displayed records after updating
            except sqlite3.Error as e:
                mb.showerror('Error', f'Failed to update record: {e}')

# Function to perform search based on the provided query
def search_record(event=None):
    query = search_entry.get()  # Get the search query from the Entry widget
    if query:
        # Clear the Treeview of any previous search results
        tree.delete(*tree.get_children())
        # Execute the SQL query to find records matching the search query
        cursor.execute("SELECT * FROM SCHOOL_MANAGEMENT WHERE NAME LIKE ? OR EMAIL LIKE ? OR PHONE_NO LIKE ?",
                       ('%' + query + '%', '%' + query + '%', '%' + query + '%'))
        data = cursor.fetchall()
        # Display the search results in the Treeview
        for records in data:
            tree.insert('', END, values=records)
    else:
        # If the search query is empty, display all records
        display_records()

def choose_image():
    def create_school_images_table():
        try:
            connector = mysql.connector.connect(
                host="localhost",
                user="root",
                password="baby1",
                database="SchoolManagement"
            )
            cursor = connector.cursor()

            # SQL query to create SCHOOL_IMAGES table
            create_table_query = """
            CREATE TABLE SCHOOL_IMAGES (
                ID INT AUTO_INCREMENT PRIMARY KEY,
                REGISTRATION_NUMBER VARCHAR(50) NOT NULL,
                IMAGE LONGBLOB NOT NULL
            )
            """
            cursor.execute(create_table_query)
            connector.commit()

        except mysql.connector.Error as e:
            mb.showerror('Error', f'Failed to create table: {e}')

        finally:
            if 'connector' in locals() and connector.is_connected():
                cursor.close()
                connector.close()

    def save_image_to_database(registration_number, file_path):
        try:
            connector = mysql.connector.connect(
                host="localhost",
                user="root",
                password="baby1",
                database="SchoolManagement"
            )
            cursor = connector.cursor()

            with open(file_path, "rb") as file:
                image_data = file.read()
                cursor.execute("INSERT INTO SCHOOL_IMAGES (REGISTRATION_NUMBER, IMAGE) VALUES (%s, %s)",
                               (registration_number, image_data))
                connector.commit()
                mb.showinfo('Success', f'Image for registration number {registration_number} saved successfully.')

        except mysql.connector.Error as e:
            mb.showerror('Error', f'Failed to save image: {e}')

        finally:
            if 'connector' in locals() and connector.is_connected():
                cursor.close()
                connector.close()

    def ask_for_image_registration():
        registration_number = registration_number_strvar.get()

        if not registration_number:
            mb.showerror('Error', 'Please enter the registration number first.')
        else:
            file_path = filedialog.askopenfilename()
            if file_path:
                save_image_to_database(registration_number, file_path)
            else:
                mb.showwarning('Warning', 'No image file selected.')

    # Creating the SCHOOL_IMAGES table before saving an image
    create_school_images_table()

    # Prompting for registration number and image selection
    ask_for_image_registration()
main = Tk()
main.title('School Management System')
main.geometry('1000x600')
main.resizable(0, 0)
main.configure(bg='blue')

lf_bg = 'lightblue'
cf_bg = 'lightblue'

name_strvar = StringVar()
email_strvar = StringVar()
contact_strvar = StringVar()
gender_strvar = StringVar()

stream_strvar = StringVar()
address_strvar = StringVar()
grade_strvar = StringVar()
registration_number_strvar = StringVar()
school_strvar = StringVar()
department_strvar = StringVar()


def initialize_fields():
    name_strvar.set('')
    email_strvar.set('')
    contact_strvar.set('')
    gender_strvar.set('')
    stream_strvar.set('')
    grade_strvar.set('')
    dob = datetime.datetime.now().date()  # Update the dob variable directly
    registration_number_strvar.set('')
    school_strvar.set('')
    department_strvar.set("")
    address_strvar.set('')
# Define student details label
student_details_label = Label(main, text="MMUST STUDENT DETAILS", font=('Arial', 14, 'bold'))
student_details_label.grid(row=0, column=0, padx=30, pady=5, sticky='nw')

# Define search_frame
search_frame = Frame(main, width=200, height=100,bg='lightblue')
search_frame.grid(row=1, column=0, padx=30, pady=5, sticky='nw')

# Define left_frame
left_frame = Frame(main, width=300, height=300, bg='lightblue')
left_frame.grid(row=2, column=0, padx=30, pady=5, sticky='nw')

# Define right_frame
right_frame = Frame(main, width=400, height=380, bg='blue')
right_frame.grid(row=0, column=1, rowspan=3, padx=20, pady=10, sticky='nsew')

# Create a Canvas widget within the right_frame using grid
canvas_width = 400
canvas_height = 380
canvas = Canvas(right_frame, width=canvas_width, height=canvas_height, bg='blue')
canvas.grid(row=0, column=0, padx=10, pady=30, sticky='nsew')

# Define center_frame
center_frame = Frame(main, width=100, height=100, bg='blue')
center_frame.grid(row=3, column=1, padx=5, pady=5, sticky='nsew')

# Configuration for grid resizing
main.grid_rowconfigure(0, weight=1)
main.grid_rowconfigure(1, weight=1)
main.grid_rowconfigure(2, weight=1)
main.grid_rowconfigure(3, weight=1)
main.grid_columnconfigure(0, weight=1)
main.grid_columnconfigure(1, weight=1)

# DateEntry widget for date of birth
dob = DateEntry(left_frame, font=("Arial", 10), width=15)
dob.grid(row=4, column=1, padx=5, pady=5, sticky='w')
dob.set_date(datetime.datetime.now().date().strftime("%m/%d/%Y"))

# Labels and Entry fields for the left frame
Label(left_frame, text="Name", font=labelfont, bg=lf_bg).grid(row=0, column=0, padx=5, pady=5, sticky='w')
Entry(left_frame, width=19, textvariable=name_strvar, font=entryfont).grid(row=0, column=1, padx=5, pady=5, sticky='w')

Label(left_frame, text="Contact Number", font=labelfont, bg=lf_bg).grid(row=1, column=0, padx=5, pady=5, sticky='w')
Entry(left_frame, width=19, textvariable=contact_strvar, font=entryfont).grid(row=1, column=1, padx=5, pady=5, sticky='w')

Label(left_frame, text="Email Address", font=labelfont, bg=lf_bg).grid(row=2, column=0, padx=5, pady=5, sticky='w')
Entry(left_frame, width=19, textvariable=email_strvar, font=entryfont).grid(row=2, column=1, padx=5, pady=5, sticky='w')

# Create the OptionMenu
gender_option_menu = OptionMenu(left_frame, gender_strvar, 'Male', 'Female')
gender_option_menu.config(width=15, font=('Arial', 10))  # Set width and font
gender_option_menu.grid(row=3, column=1, padx=5, pady=5, sticky='w')  # Place the OptionMenu

Label(left_frame, text="Date of Birth (DOB)", font=labelfont, bg=lf_bg).grid(row=4, column=0, padx=5, pady=5, sticky='w')
DateEntry(left_frame, font=("Arial", 12), width=15, textvariable=dob).grid(row=4, column=1, padx=5, pady=5, sticky='w')

Label(left_frame, text="Stream", font=labelfont, bg=lf_bg).grid(row=5, column=0, padx=5, pady=5, sticky='w')
Entry(left_frame, width=19, textvariable=stream_strvar, font=entryfont).grid(row=5, column=1, padx=5, pady=5, sticky='w')

Label(left_frame, text="Registration Number", font=labelfont, bg=cf_bg).grid(row=6, column=0, padx=5, pady=5, sticky='w')
Entry(left_frame, width=19, textvariable=registration_number_strvar, font=entryfont).grid(row=6, column=1, padx=5, pady=5, sticky='w')

Label(left_frame, text="School", font=labelfont, bg=cf_bg).grid(row=7, column=0, padx=5, pady=5, sticky='w')
Entry(left_frame, width=19, textvariable=school_strvar, font=entryfont).grid(row=7, column=1, padx=5, pady=5, sticky='w')

Label(left_frame, text="Department", font=labelfont, bg=cf_bg).grid(row=8, column=0, padx=5, pady=5, sticky='w')
Entry(left_frame, width=19, textvariable=department_strvar, font=entryfont).grid(row=8, column=1, padx=5, pady=5, sticky='w')

Label(left_frame, text="Address", font=labelfont, bg=lf_bg).grid(row=9, column=0, padx=5, pady=5, sticky='w')
Entry(left_frame, width=19, textvariable=address_strvar, font=entryfont).grid(row=9, column=1, padx=5, pady=5, sticky='w')

Label(left_frame, text="Grade", font=labelfont, bg=lf_bg).grid(row=10, column=0, padx=5, pady=5, sticky='w')
Entry(left_frame, width=19, textvariable=grade_strvar, font=entryfont).grid(row=10, column=1, padx=5, pady=5, sticky='w')



Button(center_frame, text='Submit', font=('Arial', 8), command=add_record, width=12, bg="orange").grid(row=3, column=1, padx=3, pady=3)
Button(center_frame, text='View Record', font=('Arial', 8), command=view_record, width=12, bg="orange").grid(row=3, column=2, padx=3, pady=3)
Button(center_frame, text='Reset Fields', font=('Arial', 8), command=initialize_fields, width=12, bg="orange").grid(row=3, column=3, padx=3, pady=3)
Button(center_frame, text='Delete Record', font=('Arial', 8), command=remove_record, width=12, bg="orange").grid(row=3, column=4, padx=3, pady=3)
# Create a toggle button
toggle_button = Button(center_frame, text='Clear Window', font=('Arial', 8), command=toggle_clear_records, width=12,bg="orange")
toggle_button.grid(row=3, column=5, padx=3, pady=3)
Button(center_frame, text='Edit Record', font=('Arial', 8), command=edit_record, width=12, bg="orange").grid(row=3, column=6, padx=3, pady=3)
# Create the Choose Image button in center_frame
Button(center_frame, text='Choose Image', font=('Arial', 8), command=choose_image, width=12,bg="orange").grid(row=3, column=7, padx=3, pady=3)
# Create Search Entry and Button within search_frame
search_entry = Entry(search_frame, font=('Arial', 10),width=35)
search_entry.grid(row=0, column=0, padx=5, pady=5)
search_entry.bind('<KeyRelease>', search_record)

search_button = Button(search_frame, text='Search', bg="orange",font=('Arial', 10), command=search_record)
search_button.grid(row=0, column=1, padx=5, pady=5)


# Create a ttk style object
style = ttk.Style()

# Configure a custom style for Treeview headings
style.theme_use("default")  # Use the default theme
style.configure("Treeview.Heading", font=('Arial', 12, 'bold'), foreground="white", background="black")  # Modify the font, foreground (text color), and background color

# Create and configure the Treeview widget
tree = ttk.Treeview(right_frame, height=100, selectmode="browse",
                    columns=('Serial/No', 'Name', 'Email', 'Contact', 'Gender', 'Date of Birth', 'Stream', 'Grade', 'Registration No', 'School', 'Department', 'Address'), show="headings")
tree.grid(row=0, column=0, padx=5, pady=5, sticky='nsew', columnspan=3)

# Define the Scrollbars
X_scroller = Scrollbar(right_frame, orient=HORIZONTAL, command=tree.xview)
Y_scroller = Scrollbar(right_frame, orient=VERTICAL, command=tree.yview)

# Apply grid for Treeview, Scrollbars, and configure the Treeview
tree.grid(row=0, column=0, padx=5, pady=5, sticky='nsew')  # Treeview
X_scroller.grid(row=1, column=0, sticky='ew')  # X Scroller
Y_scroller.grid(row=0, column=1, sticky='ns')  # Y Scroller
# Allow scrollbars to stretch with the window
right_frame.grid_rowconfigure(0, weight=1)
right_frame.grid_columnconfigure(0, weight=1)
X_scroller.grid(row=1, column=0, sticky='ew')
Y_scroller.grid(row=0, column=1, sticky='ns')

# Place the treeview inside the right_frame
tree.place(y=30, relwidth=1, relheight=1, relx=0, rely=0)

# Configure Treeview with Scrollbars
tree.config(xscrollcommand=X_scroller.set, yscrollcommand=Y_scroller.set)

tree.config(xscrollcommand=X_scroller.set, yscrollcommand=Y_scroller.set)


# Display headings
tree.heading('Serial/No', text='No', anchor=CENTER)
tree.heading('Name', text='Name', anchor=CENTER)
tree.heading('Email', text='Email Address', anchor=CENTER)
tree.heading('Contact', text='Contact Number', anchor=CENTER)
tree.heading('Gender', text='Gender', anchor=CENTER)
tree.heading('Date of Birth', text='Date of Birth', anchor=CENTER)
tree.heading('Stream', text='Stream', anchor=CENTER)
tree.heading('Grade', text='Grade', anchor=CENTER)
tree.heading('Registration No', text='Registration No', anchor=CENTER)
tree.heading('School', text='School', anchor=CENTER)
tree.heading('Department', text='Department', anchor=CENTER)
tree.heading('Address', text='Address', anchor=CENTER)

# Set column widths and stretch properties
tree.column('Serial/No', width=50, stretch=NO)
tree.column('Name', width=150, stretch=NO)
tree.column('Email', width=150, stretch=NO)
tree.column('Contact', width=100, stretch=NO)
tree.column('Gender', width=80, stretch=NO)
tree.column('Date of Birth', width=80, stretch=NO)
tree.column('Stream', width=80, stretch=NO)
tree.column('Grade', width=80, stretch=NO)
tree.column('Registration No', width=150, stretch=NO)
tree.column('School', width=120, stretch=NO)
tree.column('Department', width=100, stretch=NO)
tree.column('Address', width=150, stretch=NO)


tree.place(y=30, relwidth=1, relheight=0.9, relx=0)

display_records()
main.mainloop()




