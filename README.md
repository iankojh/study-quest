import time
import random
import sys

class StudyQuestApp:
    def __init__(self):
        # --- Game state ---
        self.tasks = []
        self.xp = 0
        self.streak = 0
        self.day = 1
        self.current_user = None
        self.logged_in = False

        # --- Users (in-memory for now) ---
        self.users = {
            "bloomer": {"password": "yo", "xp": 100, "streak": 80085}
        }

        # --- Rewards ---
        self.rewards = [
            {"name": "10 mins break", "xp_cost": 15},
            {"name": "1 episode of anime", "xp_cost": 30},
            {"name": "1 hour gaming", "xp_cost": 60},
            {"name": "Custom free time", "xp_cost": None}
        ]

        # --- Inactivity + timeout control ---
        self.last_active = time.time()
        self.timeout = 120  # seconds of inactivity before auto-logout
        self.retry_count = 0
        self.max_retries = 3

        # --- Login attempt control ---
        self.failed_login_attempts = 0
        self.max_login_attempts = 5

    # ============================================================
    # Utility: all inputs go here
    # ============================================================
    def get_input(self, prompt: str) -> str:
        self.check_inactivity()
        value = input(prompt)
        self.reset_activity()
        return value

    def reset_activity(self):
        self.last_active = time.time()

    def check_inactivity(self):
        """If logged in and idle too long, save, log out, and count a timeout."""
        if self.logged_in and (time.time() - self.last_active > self.timeout):
            print("\n⏰ You were inactive too long. Logging out...")
            self.save_user_data()
            self.logged_in = False
            self.current_user = None
            self.handle_timeout(reason="inactivity")

    def handle_timeout(self, reason: str):
        """Count a timeout; exit if too many."""
        self.retry_count += 1
        print(f"⚠️ Timeout triggered due to {reason}. ({self.retry_count}/{self.max_retries})")
        if self.retry_count >= self.max_retries:
            print("❌ Too many timeouts. Exiting StudyQuest.")
            sys.exit(0)

    # ============================================================
    # Login / Register
    # ============================================================
    def log_in(self):
        print("=== Welcome to StudyQuest! ===")
        self.failed_login_attempts = 0
        while not self.logged_in:
            username = self.get_input("Username (or 'register'): ").strip()
            if username.lower() == "register":
                self.register_user()
                continue

            password = self.get_input("Password: ").strip()

            if username in self.users and self.users[username]["password"] == password:
                print(f"✅ Login successful! Welcome, {username}.")
                self.logged_in = True
                self.current_user = username
                self.xp = self.users[username]["xp"]
                self.streak = self.users[username]["streak"]
                self.failed_login_attempts = 0
                self.reset_activity()
            else:
                self.failed_login_attempts += 1
                print(f"❌ Invalid username or password. Attempts: {self.failed_login_attempts}/{self.max_login_attempts}")
                if self.failed_login_attempts >= self.max_login_attempts:
                    print("⏳ Too many failed login attempts. Timing out...")
                    self.handle_timeout(reason="too many login failures")
                    self.failed_login_attempts = 0

    def register_user(self):
        print("\n--- Register New User ---")
        while True:
            new_username = self.get_input("Choose a username: ").strip()
            if not new_username:
                print("⚠️ Username cannot be empty.")
                continue
            if new_username in self.users:
                print("⚠️ Username already exists. Try again.")
                continue

            new_password = self.get_input("Choose a password: ").strip()
            if not new_password:
                print("⚠️ Password cannot be empty.")
                continue

            try:
                starting_xp_str = self.get_input("Set starting XP (number): ").strip()
                starting_xp = int(starting_xp_str) if starting_xp_str else 0
            except ValueError:
                starting_xp = 0

            self.users[new_username] = {"password": new_password, "xp": starting_xp, "streak": 0}
            print(f"✅ User '{new_username}' registered with {starting_xp} XP.")
            break

    # ============================================================
    # Program lifecycle
    # ============================================================
    def run(self):
        self.load_user_data()
        self.log_in()

        while True:
            self.check_inactivity()
            self.show_main_menu()
            choice = self.get_input("Select an option (1-6): ").strip()

            if choice == "1":
                if not self.check_today_tasks():
                    self.create_todo_list()
                while self.tasks_remaining():
                    self.check_inactivity()
                    if not self.run_task_cycle():
                        break
                self.show_daily_summary()

            elif choice == "2":
                self.display_stats()

            elif choice == "3":
                self.show_rewards()
                self.redeem_reward()

            elif choice == "4":
                self.show_today_tasks()

            elif choice == "5":
                self.advance_day()

            elif choice == "6":
                print("Goodbye! Progress saved.")
                self.save_user_data()
                break

            else:
                print("Invalid option. Try again.")

    def load_user_data(self):
        print("Loading StudyQuest... (user data is in-memory for this demo)")

    def save_user_data(self):
        if self.current_user:
            self.users[self.current_user]["xp"] = self.xp
            self.users[self.current_user]["streak"] = self.streak

    # ============================================================
    # Menus & Task Flow
    # ============================================================
    def show_main_menu(self):
        print("\n--- StudyQuest Main Menu ---")
        print("1. Start/Continue Tasks")
        print("2. View XP and Streak")
        print("3. Redeem Rewards")
        print("4. View Today's Tasks")
        print("5. Advance to Next Day")
        print("6. Exit")

    def check_today_tasks(self):
        return bool(self.get_today_tasks())

    def get_today_tasks(self):
        return self.tasks

    def create_todo_list(self):
        print("Create your To-Do List for today:")
        num_tasks = 3
        for i in range(num_tasks):
            while True:
                task_name = self.get_input(f"Enter task {i+1} name: ").strip()
                if not task_name:
                    print("Task name cannot be empty, try again.")
                    continue
                if any(t['name'].lower() == task_name.lower() for t in self.tasks):
                    print("Task already exists, please enter a unique task.")
                    continue
                if task_name == 'goon':  # Easter egg
                    print('67')
                    self.xp += 1000000
                break
            task = {"name": task_name, "xp": 10, "completed": False}
            self.tasks.append(task)
        print("Tasks saved!")

    def tasks_remaining(self):
        return any(not task["completed"] for task in self.tasks)

    def run_task_cycle(self):
        task = self.get_next_task()
        if not task:
            return False

        print(f"\nStarting task: {task['name']}")
        stop = self.get_input("Press Enter to begin or type 'stop' to return to menu: ").strip().lower()
        if stop == 'stop':
            print("Returning to menu...")
            return False

        if self.start_focus_timer(task):
            self.complete_task(task)
            self.reward_user(task)
        else:
            self.remind_focus()
        return True

    def get_next_task(self):
        for task in self.tasks:
            if not task["completed"]:
                return task
        return None

    def start_focus_timer(self, task):
        print("Focus Timer: 25 minutes (simulated 3s)...")
        time.sleep(3)
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
        return self.xp > 0 and self.xp % 30 == 0

    def unlock_mini_game(self):
        print("🎮 Mini-game unlocked! Take a short break and play!")

    def send_motivational_message(self):
        messages = [
            "You're doing great! Keep going! 💪",
            "Every step counts. 📈",
            "Focus now, relax later. 🧘",
            "Success is built one task at a time! 🧱"
        ]
        if random.randint(0, 1000) >= 969:  # rare bonus
            print("You got this twin. Keep going twin on some nonchalant shi")
            print("💥 30 XP bonus!")
            self.xp += 30
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

    def display_stats(self):
        print(f"\n📊 Your Stats: XP = {self.xp}, Streak = {self.streak}, Day = {self.day}")

    # ============================================================
    # Rewards
    # ============================================================
    def show_rewards(self):
        print("\n🎁 Available Rewards:")
        for i, reward in enumerate(self.rewards, 1):
            cost_text = "{} XP".format(reward["xp_cost"]) if reward["xp_cost"] is not None else "Varies"
            print(f"{i}. {reward['name']} (Cost: {cost_text})")

    def redeem_reward(self):
        choice = self.get_input("Enter reward number to redeem or press Enter to skip: ").strip()
        if choice == 'goon':  # Easter egg
            print("67 hours of gooning redeemed")
            return
        if not choice:
            return
        if not choice.isdigit():
            print("Invalid choice.")
            return

        choice_num = int(choice)
        if choice_num < 1 or choice_num > len(self.rewards):
            print("Invalid choice.")
            return

        selected = self.rewards[choice_num - 1]

        if selected["name"] == "Custom free time":
            try:
                hours_str = self.get_input("Enter number of hours of free time: ").strip()
                hours = int(hours_str)
                cost = hours * 20
                if self.xp >= cost:
                    self.xp -= cost
                    print(f"✅ You redeemed: {hours} hours of free time! Remaining XP: {self.xp}")
                else:
                    print(f"❌ Not enough XP to redeem {hours} hours (cost: {cost} XP).")
            except ValueError:
                print("Invalid number.")
            return

        if self.xp >= selected["xp_cost"]:
            self.xp -= selected["xp_cost"]
            print(f"✅ You redeemed: {selected['name']}! Remaining XP: {self.xp}")
        else:
            print("❌ Not enough XP to redeem this reward.")

    # ============================================================
    # Day Management
    # ============================================================
    def show_today_tasks(self):
        if not self.tasks:
            print("No tasks added today.")
            return
        print("\n📋 Today's Tasks:")
        for task in self.tasks:
            status = "✅" if task["completed"] else "❌"
            print(f"{status} {task['name']}")

    def advance_day(self):
        self.day += 1
        self.tasks.clear()
        print(f"\n📅 Advanced to Day {self.day}. Task list cleared.")


if __name__ == "__main__":
    app = StudyQuestApp()
    app.run()
