import time
import random

class StudyQuestApp:
    def __init__(self):
        self.tasks = []
        self.xp = 0
        self.streak = 0

    def run(self):
        self.load_user_data()
        if not self.check_today_tasks():
            self.create_todo_list()
        while self.tasks_remaining():
            self.run_task_cycle()
        self.show_daily_summary()

    def load_user_data(self):
        print("Welcome back to StudyQuest!")
        # In future: load user profile, XP, streaks, and previous tasks from file/db

    def check_today_tasks(self):
        print("Checking today's tasks...")
        return bool(self.get_today_tasks())

    def get_today_tasks(self):
        # In future: query from file/db. Now, just use current list.
        return self.tasks

    def create_todo_list(self):
        print("Create your To-Do List for today:")
        num_tasks = 3
        for i in range(num_tasks):
            while True:
                task_name = input(f"Enter task {i+1} name: ").strip()
                if not task_name:
                    print("Task name cannot be empty, try again.")
                    continue
                if any(t['name'].lower() == task_name.lower() for t in self.tasks):
                    print("Task already exists, please enter a unique task.")
                    continue
                break
            task = {"name": task_name, "xp": 10, "completed": False}
            self.tasks.append(task)
        print("Tasks saved!")

    def tasks_remaining(self):
        return any(not task["completed"] for task in self.tasks)

    def run_task_cycle(self):
        task = self.get_next_task()
        if not task:
            return
        print(f"\nStarting task: {task['name']}")
        if self.start_focus_timer(task):
            self.complete_task(task)
            self.reward_user(task)
        else:
            self.remind_focus()

    def get_next_task(self):
        for task in self.tasks:
            if not task["completed"]:
                return task
        return None

    def start_focus_timer(self, task):
        print("Focus Timer: 25 minutes (simulated 3s)...")
        time.sleep(3)
        # Simulate focus success
        return True

    def complete_task(self, task):
        task["completed"] = True
        print(f"Task '{task['name']}' completed!")

    def reward_user(self, task):
        self.xp += task["xp"]
        self.streak += 1
        print(f"+{task['xp']} XP! Total XP: {self.xp}, Streak: {self.streak}")
        if self.xp_threshold_reached():
            self.unlock_mini_game()
        else:
            self.send_motivational_message()

    def xp_threshold_reached(self):
        return self.xp > 0 and self.xp % 30 == 0  # Mini-game unlock every 30 XP

    def unlock_mini_game(self):
        print("🎮 Mini-game unlocked! Take a short break and play!")

    def send_motivational_message(self):
        messages = [
            "You're doing great! Keep going! 💪",
            "Every step counts. 📈",
            "Focus now, relax later. 🧘",
            "Success is built one task at a time! 🧱"
        ]
        print("💬 " + random.choice(messages))

    def remind_focus(self):
        print("⚠️ Reminder: Try to stay focused. You got this!")

    def show_daily_summary(self):
        print("\n--- Daily Summary ---")
        completed = sum(1 for task in self.tasks if task["completed"])
        print(f"Tasks completed: {completed}/{len(self.tasks)}")
        print(f"Total XP: {self.xp}")
        print(f"Current streak: {self.streak}")
        self.celebrate_user()

    def celebrate_user(self):
        print("\n🎉 Great work today! See you tomorrow on StudyQuest!")

if __name__ == "__main__":
    app = StudyQuestApp()
    app.run()
