You're right—currently your `update_task` and `delete_task` methods iterate through both `day_tasks` and `night_tasks` separately, duplicating logic. Also, in your `main.py`, the update prompt asks for new input even if the task ID is invalid.

Here's how we’ll improve this:

---

### **Improvements in `task_manager.py`:**

1. **Combine the search logic into a helper method `find_task_by_id`** to avoid repeated code.
2. **Only proceed with update if the task is found.**
3. **Update the task in-place instead of removing and appending.**

### **Improved `task_manager.py`**

```python
from model.task import Task
from datetime import time

class TaskManager:
    def __init__(self):
        self.tasks = {"day_tasks": [], "night_tasks": []}

    def add_task(self, task: Task):
        try:
            if len(self.tasks["day_tasks"]) >= 10 and len(self.tasks["night_tasks"]) >= 10:
                return "You have added the maximum number of tasks."

            task_time = task.date_time_to_do.time()

            if time(6, 0) <= task_time <= time(21, 0):
                if len(self.tasks["day_tasks"]) < 10:
                    self.tasks["day_tasks"].append(task)
                    return "Task added to day tasks successfully."
                else:
                    return "You have added the maximum number of day tasks."
            else:
                if len(self.tasks["night_tasks"]) < 10:
                    self.tasks["night_tasks"].append(task)
                    return "Task added to night tasks successfully."
                else:
                    return "You have added the maximum number of night tasks."
        except Exception as e:
            return f"Error Adding Task: {e}"

    def view_tasks(self):
        return self.tasks

    def find_task_by_id(self, task_id):
        for key in ["day_tasks", "night_tasks"]:
            for idx, task in enumerate(self.tasks[key]):
                if task.id == task_id:
                    return key, idx
        return None, None

    def update_task(self, task_id: int, updated_task: Task):
        try:
            key, idx = self.find_task_by_id(task_id)
            if key is not None:
                self.tasks[key][idx] = updated_task
                return f"Task {task_id} updated successfully."
            return f"Task {task_id} not found."
        except Exception as e:
            return f"Error Updating Task: {e}"

    def delete_task(self, task_id: int):
        try:
            key, idx = self.find_task_by_id(task_id)
            if key is not None:
                self.tasks[key].pop(idx)
                return f"Task {task_id} is deleted."
            return f"Task {task_id} not found."
        except Exception as e:
            return f"Error Deleting Task: {e}"
```

---

### **Improvements in `main.py`:**

* **Ask for update details only after confirming the ID exists.**

```python
elif option == "U":
    task_id = int(input("Enter Task ID to update: "))
    found_key, _ = task_manager.find_task_by_id(task_id)
    if found_key is None:
        print(f"Task {task_id} not found.")
    else:
        title = input("Enter Title of Task: ")
        description = input("Enter Description of the Task: ")
        date_time_to_do = input("Enter Task Time (YYYY-MM-DD HH:MM): ")
        updated_task = Task(id=task_id, title=title, description=description, date_time_to_do=date_time_to_do)
        result = task_manager.update_task(task_id, updated_task)
        print(result)
```

---

This approach:

* Removes duplicate looping logic.
* Avoids asking for unnecessary input.
* Improves efficiency and readability.

Would you like me to update the entire code structure with this refactoring included?
