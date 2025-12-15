# technical-training
project
import json
import uuid
from datetime import datetime, timedelta
from colorama import Fore, Style
from pathlib import Path
import sys

DB = Path("tasks.json")

# ---------------------------
# Load & Save
# ---------------------------

def load_tasks():
    if not DB.exists():
        return []
    return json.loads(DB.read_text())

def save_tasks(tasks):
    DB.write_text(json.dumps(tasks, indent=4))

# ---------------------------
# Add Task
# ---------------------------

def add_task(title, deadline, priority="Medium", description=""):
    tasks = load_tasks()

    task = {
        "id": str(uuid.uuid4())[:8],
        "title": title,
        "description": description,
        "priority": priority,
        "deadline": deadline,
        "completed": False,
        "created_at": datetime.now().isoformat()
    }

    tasks.append(task)
    save_tasks(tasks)
    print(Fore.GREEN + "Task added successfully!" + Style.RESET_ALL)

# ---------------------------
# Update Task
# ---------------------------

def update_task(task_id, new_title=None, new_deadline=None, new_priority=None):
    tasks = load_tasks()
    found = False

    for t in tasks:
        if t["id"] == task_id:
            found = True
            if new_title:
                t["title"] = new_title
            if new_deadline:
                t["deadline"] = new_deadline
            if new_priority:
                t["priority"] = new_priority

    save_tasks(tasks)

    if found:
        print(Fore.GREEN + "Task updated successfully!" + Style.RESET_ALL)
    else:
        print(Fore.RED + "Task ID not found." + Style.RESET_ALL)

# ---------------------------
# Delete Task
# ---------------------------

def delete_task(task_id):
    tasks = load_tasks()
    new_tasks = [t for t in tasks if t["id"] != task_id]
    save_tasks(new_tasks)

    print(Fore.YELLOW + "Task deleted!" + Style.RESET_ALL)

# ---------------------------
# Mark Completed
# ---------------------------

def complete_task(task_id):
    tasks = load_tasks()
    for t in tasks:
        if t["id"] == task_id:
            t["completed"] = True
            save_tasks(tasks)
            print(Fore.GREEN + "Task marked completed!" + Style.RESET_ALL)
            return

    print(Fore.RED + "Task ID not found." + Style.RESET_ALL)

# ---------------------------
# List Tasks
# ---------------------------

def list_tasks():
    tasks = load_tasks()

    if not tasks:
        print(Fore.RED + "No tasks found." + Style.RESET_ALL)
        return

    now = datetime.now()

    # Sort by deadline then priority
    def sort_key(t):
        d = datetime.fromisoformat(t["deadline"])
        p = {"High": 0, "Medium": 1, "Low": 2}[t["priority"]]
        return (d, p)

    tasks = sorted(tasks, key=sort_key)

    print(Fore.CYAN + "\n==== YOUR TASKS ====\n" + Style.RESET_ALL)

    for t in tasks:
        deadline = datetime.fromisoformat(t["deadline"])
        hours_left = (deadline - now).total_seconds() / 3600

        color = Fore.RED if hours_left < 24 and not t["completed"] else Fore.GREEN
        status = "✔" if t["completed"] else "✖"

        print(
            color +
            f"[{status}] {t['id']} | {t['title']} | {t['priority']} | "
            f"Due: {t['deadline']}" +
            Style.RESET_ALL
        )

# ---------------------------
# CLI COMMAND HANDLER
# ---------------------------

def main():
    if len(sys.argv) < 2:
        print("Commands: add | update | delete | done | list")
        return

    command = sys.argv[1]

    if command == "add":
        title = sys.argv[2]
        deadline = sys.argv[3]       # Format: YYYY-MM-DDTHH:MM
        priority = sys.argv[4] if len(sys.argv) > 4 else "Medium"
        description = sys.argv[5] if len(sys.argv) > 5 else ""
        add_task(title, deadline, priority, description)

    elif command == "update":
        task_id = sys.argv[2]
        new_title = sys.argv[3] if len(sys.argv) > 3 else None
        new_deadline = sys.argv[4] if len(sys.argv) > 4 else None
        new_priority = sys.argv[5] if len(sys.argv) > 5 else None
        update_task(task_id, new_title, new_deadline, new_priority)

    elif command == "delete":
        delete_task(sys.argv[2])

    elif command == "done":
        complete_task(sys.argv[2])

    elif command == "list":
        list_tasks()

    else:
        print("Invalid command.")

if __name__ == "__main__":
    main()
