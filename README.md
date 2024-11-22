import tkinter as tk
from tkinter import ttk, messagebox
import psycopg2

# Kết nối cơ sở dữ liệu PostgreSQL
def connect_db():
    try:
        conn = psycopg2.connect(
            host="localhost",  
            database="BaiTapDuAn2",
            user="postgres",
            password="161024"
        )
        return conn
    except Exception as e:
        messagebox.showerror("Lỗi cơ sở dữ liệu", str(e))
        return None

# Các hàm thao tác với cơ sở dữ liệu
def fetch_students():
    conn = connect_db()
    if conn:
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM students")
        rows = cursor.fetchall()
        conn.close()
        return rows
    return []

def insert_student(name, age, gender, major):
    conn = connect_db()
    if conn:
        cursor = conn.cursor()
        cursor.execute(
            "INSERT INTO students (name, age, gender, major) VALUES (%s, %s, %s, %s)",
            (name, age, gender, major)
        )
        conn.commit()
        conn.close()

def update_student(student_id, name, age, gender, major):
    conn = connect_db()
    if conn:
        cursor = conn.cursor()
        cursor.execute(
            "UPDATE students SET name=%s, age=%s, gender=%s, major=%s WHERE id=%s",
            (name, age, gender, major, student_id)
        )
        conn.commit()
        conn.close()

def delete_student(student_id):
    conn = connect_db()
    if conn:
        cursor = conn.cursor()
        cursor.execute("DELETE FROM students WHERE id=%s", (student_id,))
        conn.commit()
        conn.close()

# Lớp ứng dụng chính
class StudentApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Quản lý Sinh Viên")
        self.root.geometry("700x500")  # Đặt kích thước cửa sổ
        self.root.configure(bg="#E9F5FF")  # Đặt màu nền

        # Biến lưu trữ thông tin nhập liệu
        self.name_var = tk.StringVar()
        self.age_var = tk.StringVar()
        self.gender_var = tk.StringVar()
        self.major_var = tk.StringVar()

        # Form nhập liệu
        form_frame = tk.Frame(self.root, bg="#E9F5FF", relief=tk.RAISED, bd=2)
        form_frame.pack(pady=10, padx=20, fill=tk.X)

        tk.Label(form_frame, text="Tên:", bg="#E9F5FF", font=("Times New Roman", 12)).grid(row=0, column=0, padx=10, pady=5)
        tk.Entry(form_frame, textvariable=self.name_var, font=("Times New Roman", 12)).grid(row=0, column=1, pady=5, sticky=tk.W)

        tk.Label(form_frame, text="Tuổi:", bg="#E9F5FF", font=("Times New Roman", 12)).grid(row=0, column=2, padx=10, pady=5)
        tk.Entry(form_frame, textvariable=self.age_var, font=("Times New Roman", 12)).grid(row=0, column=3, pady=5, sticky=tk.W)

        tk.Label(form_frame, text="Giới tính:", bg="#E9F5FF", font=("Times New Roman", 12)).grid(row=1, column=0, padx=10, pady=5)
        tk.Entry(form_frame, textvariable=self.gender_var, font=("Times New Roman", 12)).grid(row=1, column=1, pady=5, sticky=tk.W)

        tk.Label(form_frame, text="Ngành học:", bg="#E9F5FF", font=("Times New Roman", 12)).grid(row=1, column=2, padx=10, pady=5)
        tk.Entry(form_frame, textvariable=self.major_var, font=("Times New Roman", 12)).grid(row=1, column=3, pady=5, sticky=tk.W)

        # Các nút chức năng
        button_frame = tk.Frame(self.root, bg="#E9F5FF")
        button_frame.pack(pady=10)

        tk.Button(button_frame, text="Thêm sinh viên", command=self.add_student, bg="#4CAF50", fg="white", font=("Times New Roman", 12), width=15).grid(row=0, column=0, padx=10)
        tk.Button(button_frame, text="Cập nhật thông tin", command=self.update_student, bg="#2196F3", fg="white", font=("Times New Roman", 12), width=15).grid(row=0, column=1, padx=10)
        tk.Button(button_frame, text="Xóa sinh viên", command=self.delete_student, bg="#f44336", fg="white", font=("Times New Roman", 12), width=15).grid(row=0, column=2, padx=10)
        tk.Button(button_frame, text="Tải lại danh sách", command=self.load_students, bg="light pink", fg="white", font=("Times New Roman", 12), width=15).grid(row=0, column=3, padx=10)

        # Bảng hiển thị danh sách sinh viên
        self.tree = ttk.Treeview(self.root, columns=("ID", "Name", "Age", "Gender", "Major"), show="headings", height=10)

        # Điều chỉnh chiều rộng cột
        self.tree.column("ID", width=50)  # Điều chỉnh chiều rộng cho cột ID
        self.tree.column("Name", width=150)  # Điều chỉnh chiều rộng cho cột Tên
        self.tree.column("Age", width=50)  # Điều chỉnh chiều rộng cho cột Tuổi
        self.tree.column("Gender", width=80)  # Điều chỉnh chiều rộng cho cột Giới tính
        self.tree.column("Major", width=150)  # Điều chỉnh chiều rộng cho cột Ngành học

        # Tiêu đề cột
        self.tree.heading("ID", text="ID")
        self.tree.heading("Name", text="Tên")
        self.tree.heading("Age", text="Tuổi")
        self.tree.heading("Gender", text="Giới tính")
        self.tree.heading("Major", text="Ngành học")
        self.tree.pack(pady=20, padx=20)

        # Thay đổi màu sắc cho bảng
        style = ttk.Style()
        style.configure("Treeview.Heading", font=("Times New Roman", 12, "bold"), background="#007BFF", foreground="black")  # Chữ màu đen
        style.configure("Treeview", rowheight=25)

        self.load_students()

    def add_student(self):
        name = self.name_var.get()
        age = self.age_var.get()
        gender = self.gender_var.get()
        major = self.major_var.get()

        if name and age.isdigit() and gender and major:
            insert_student(name, int(age), gender, major)
            self.load_students()
        else:
            messagebox.showwarning("Lỗi nhập liệu", "Vui lòng nhập dữ liệu hợp lệ.")

    def update_student(self):
        selected_item = self.tree.selection()
        if selected_item:
            student_id = self.tree.item(selected_item, "values")[0]
            name = self.name_var.get()
            age = self.age_var.get()
            gender = self.gender_var.get()
            major = self.major_var.get()

            if name and age.isdigit() and gender and major:
                update_student(student_id, name, int(age), gender, major)
                self.load_students()
            else:
                messagebox.showwarning("Lỗi nhập liệu", "Vui lòng nhập dữ liệu hợp lệ.")
        else:
            messagebox.showwarning("Lỗi chọn sinh viên", "Vui lòng chọn sinh viên.")

    def delete_student(self):
        selected_item = self.tree.selection()
        if selected_item:
            student_id = self.tree.item(selected_item, "values")[0]
            delete_student(student_id)
            self.load_students()
        else:
            messagebox.showwarning("Lỗi chọn sinh viên", "Vui lòng chọn sinh viên.")

    def load_students(self):
        for item in self.tree.get_children():
            self.tree.delete(item)
        students = fetch_students()
        for student in students:
            self.tree.insert("", "end", values=student)

# Khởi tạo ứng dụng Tkinter
if __name__ == "__main__":
    root = tk.Tk()
    app = StudentApp(root)
    root.mainloop()
