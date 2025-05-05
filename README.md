from tkinter import *
from tkinter import messagebox
import sqlite3 as sql

def add_task():
    task_string = task_field.get()
    if len(task_string) == 0:
        messagebox.showinfo('Error', 'Field is empty.')
    else:
        tasks.append(task_string)
        the_cursor.execute('INSERT INTO tasks (title) VALUES (?)', (task_string,))
        list_update()
        task_field.delete(0, 'end')

def list_update():
    clear_list()
    for task in tasks:
        task_listbox.insert('end', task)

def delete_task():
    try:
        the_value = task_listbox.get(task_listbox.curselection())
        if the_value in tasks:
            tasks.remove(the_value)
            list_update()
            the_cursor.execute('DELETE FROM tasks WHERE title = ?', (the_value,))
    except IndexError:
        messagebox.showinfo('Error', 'No Tasks selected. Cannot delete.')

def delete_all_tasks():
    message_box = messagebox.askyesno('Delete All', 'Are you sure?')
    if message_box:
        tasks.clear()  # Clear the tasks list
        the_cursor.execute('DELETE FROM tasks')
        list_update()

def clear_list():
    task_listbox.delete(0, 'end')

def close():
    the_connection.commit()  # Commit changes before closing
    the_cursor.close()       # Close the cursor
    the_connection.close()    # Close the database connection
    guiwindow.destroy()

def retrieve_database():
    tasks.clear()  # Clear the tasks list
    for row in the_cursor.execute('SELECT title FROM tasks'):
        tasks.append(row[0])

if __name__ == "__main__":
    guiwindow = Tk()
    guiwindow.title("To-Do List")
    guiwindow.geometry("665x400+550+250")
    guiwindow.resizable(0, 0)
    guiwindow.configure(bg="#B5E5CF")

    the_connection = sql.connect('listofTasks.db')
    the_cursor = the_connection.cursor()
    the_cursor.execute('CREATE TABLE IF NOT EXISTS tasks (title TEXT)')

    tasks = []

    functions_frame = Frame(guiwindow, bg="#8EE5EE")
    functions_frame.pack(side="top", expand=True, fill="both")

    task_label = Label(functions_frame, text="TO-DO-LIST \n Enter the Task Title:",
                       font=("arial", "14", "bold"),
                       background="#EE5EAA",
                       foreground="#FF6103")
    task_label.place(x=20, y=30)

    task_field = Entry(
        functions_frame,  # Corrected from function_frame to functions_frame
        font=("Arial", "14"),
        width=42,
        foreground="black",
        background="white",
    )
    task_field.place(x=180, y=30)

    add_button = Button(
        functions_frame,  # Corrected from function_frame to functions_frame
        text="Add",
        width=15,
        bg='#D4AC0D', font=("arial", "14", "bold"),
        command=add_task,
    )
    del_button = Button(
        functions_frame,  # Corrected from function_frame to functions_frame
        text="Remove",
        width=15,
        bg='#D4AC0D', font=("arial", "14", "bold"),
        command=delete_task,
    )
    del_all_button = Button(
        functions_frame,  # Corrected from function_frame to functions_frame
        text="Delete All",
        
        command=delete_all_tasks,
    )

    exit_button = Button(
        functions_frame,  # Corrected from function_frame to functions_frame
        text="Exit/Close",
        width=52,
        bg="#D4AC0D", font=("arial", "14", "bold"),
        command=close
    )
    add_button.place(x=18, y=80)
    del_button.place(x=240, y=80)
    del_all_button.place(x=17, y=330)
    exit_button.place(x=17, y=330)

    task_listbox = Listbox(
        functions_frame,  # Corrected from function_frame to functions_frame
        width=70,
        height=9,
        font="bold",
        selectmode='SINGLE',
        background="WHITE",
        foreground="BLACK",
        selectbackground="#FF8C00",
        selectforeground="BLACK"
    )
    task_listbox.place(x=17, y=140)

    retrieve_database()
    list_update()
    guiwindow.mainloop()
