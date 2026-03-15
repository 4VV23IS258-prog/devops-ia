# Jenkins Automated Build Setup (Poll SCM)

1.  **Jenkins Setup**: Access your Jenkins dashboard.
2.  **Auto Way**: Use Jenkins' built-in automation features to trigger builds.
3.  **Poll SCM**: Configure Jenkins to check your repository for changes on a fixed schedule.
4.  **New Item**: Create a new project for this automation.
5.  **Choose Project Name**: Give your project a clear, unique name.
6.  **Select Freestyle Project**: Choose the "Freestyle project" type.
7.  **Source Code Management**: Go to the SCM section.
8.  **Choose Git**: Select **Git** as your version control system.
9.  **Enter Repo URL**: Provide the link to your remote GitHub or GitLab repository.
10. **Build Triggers**: Scroll down to the "Build Triggers" section.
11. **Poll SCM Trigger**: Check the box for **"Poll SCM"**.
12. **Enter Schedule**: Input `* * * * *` into the Schedule box to check for updates every minute.
13. **Build Steps**: Scroll down to the "Build Steps" section.
14. **Add Build Step**: Click the dropdown to add a new step.
15. **Execute Batch Command**: Select **"Execute Windows batch command"**.
16. **Enter Command**: Input your script execution command (e.g., `python your_script_name.py`).
17. **Save**: Click the **Save** button to apply the configuration.
18. **Automation Active**: Jenkins is now monitoring your repo; it will automatically build whenever it detects a new commit.
