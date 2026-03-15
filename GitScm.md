# Jenkins Automated Build Setup (ngrok Webhook)

1.  **New Item**: Start by creating a new task.
2.  **Choose Freestyle project**: Select the standard project type.
3.  **Choose a project name**: Provide a unique identifier.
4.  **Click OK**: Proceed to the configuration page.
5.  **Go to Source Code Manager**: Navigate to the source control section.
6.  **Choose Git**: Select Git as the version control system.
7.  **Enter the URL of the repo**: Provide the link to your GitHub repository.
8.  **Go to Build Triggers**: Navigate to the triggers section.
9.  **Select GitHub hook trigger for GITScm polling**: Enable Jenkins to listen for GitHub pings.
10. **Install and run ngrok**: Execute `ngrok http 8080` to create a public tunnel.
11. **Add build step**: Scroll to the build section and add a step.
12. **Execute batch commands**: Select "Execute Windows batch command".
13. **Enter the command**: Input the execution script (e.g., `python script.py`).
14. **Save and build**: Save the project and run it once manually to initialize.
15. **Configuring the webhook**: Prepare to link GitHub to your local Jenkins.
16. **Open repository in GitHub**: Navigate to your project on GitHub.
17. **Go to setting**: Click on the "Settings" tab of the repository.
18. **Click on webhooks**: Select "Webhooks" from the left sidebar.
19. **Add webhook**: Click the "Add webhook" button.
20. **Payload URL**: Enter `[ngrok generated url]/github-webhook/`.
21. **Click add url**: Confirm and save the webhook in GitHub.
22. **Make changes to project and push the code**: Commit and push code to trigger the automatic build.
