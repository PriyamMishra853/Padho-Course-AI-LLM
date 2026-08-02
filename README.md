AI Engineer Course – Setup Notes (Windows)
1. Python 3.11
Install Python 3.11.9 (64-bit)
✅ Check Add Python to PATH
Verify:
python --version
2. VS Code
Install VS Code

Enable Add to PATH

Install Extensions:

Python
Pylance
Jupyter
Verify:

code --version
3. Git
Install Git and configure:

git config --global user.name "Priyam Mishra"
git config --global user.email "balyadev9@gmail.com"
Verify:

git --version
4. UV
Install:

powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
If blocked:

Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
Verify:

uv --version
5. Windows Terminal
Install from Microsoft Store
6. GitHub
Create GitHub account
Create a new repository
7. API Keys
Groq API Key
Gemini API Key
OpenRouter API Key
Save them safely.

8. Qdrant Cloud
Create Free Cluster

Region: AWS Mumbai

Copy:

Cluster URL
API Key
9. Create Project Folder
cd C:\Users\%USERNAME%
mkdir padho-ai-engineer
cd padho-ai-engineer
code .
10. Create .env
GROQ_API_KEY=
GOOGLE_API_KEY=
OPENROUTER_API_KEY=
QDRANT_URL=
QDRANT_API_KEY=
⚠️ Never upload .env to GitHub.

11. Create .gitignore
.env
12. Create Week Folders
mkdir week1 week2 week3 week4 week5 week6 week7 week8
13. GitHub Commands
Initialize:

git init
Add files:

git add .
Commit:

git commit -m "Initial commit"
Connect Repository:

git remote add origin <repository-url>
Push:

git branch -M main
git push -u origin main
14. Verify Setup
python --version
uv --version
git --version
code --version
Common Errors
Python not found

Reinstall with Add to PATH
UV not found

Restart PowerShell
PowerShell blocked

Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
Don't push .env

Add .env to .gitignore
