# gitops-demo
#testing the rebase
gitops-demo
microservices-in-python
#some additional instructions
Installing Python 3.X
Creating Python Virtual Environments python3 -m venv microservices-in-python/venv source venv/bin/activate
Installing Python VS Code Extension
Sample Flask Application pip install Flask
Jinja templating for Dynamic Web Pages
Using Pip to Freeze Python Dependencies pip freeze > requirements.txt pip install -r requirements.txt
Building the docker image using Dockerfile docker build -t webapp:1.0 . docker run -d -p 80:5000 --name web webapp:1.0
Writing Docker Compose file
Writing Kubernetes Manifest files for the application
Creating Helm Chart help create webapp



----
Argo cd:
argocd admin initial-password -n argocd
argocd account update-password.
