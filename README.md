```bash
uvicorn main:app --host 0.0.0.0 --port 8080
```

```python
import json, os, secrets, uuid, subprocess

from fastapi.responses import HTMLResponse
from fastapi import FastAPI, Form

from typing import Optional

app = FastAPI(docs_url="/documentation", redoc_url=None)

BOOTSTRAP = """
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.min.js" integrity="sha384-QJHtvGhmr9XOIpI6YVutG+2QOK9T+ZnN4kzFN1RtK3zEFEIsxhlmWl5/YESvpZ13" crossorigin="anonymous"></script>
"""


@app.get("/")
async def root():
    html_content = f"""
        <html>
            <head>
                {BOOTSTRAP}
            </head>
            <body>
                <div class="container">
                    <div class="jumbotron jumbotron-fluid">
                        <div class="container">
                            <h1 class="display-4">Ansible Secret Encryption</h1>
                            <p class="lead">Tool for encrypt via ansible vault.</p>
                        </div>
                        <hr>
                        <div class="row">
                            <a class="btn btn-outline-info btn-block mb-4" href="/encrypt/file">Encrypt File</a>
                            <a class="btn btn-outline-info btn-block" href="/encrypt/string">Encrypt String</a>
                        </div>
                    </div>
                </div>
            </body>
        </html>
        """
    return HTMLResponse(content=html_content, status_code=200)


@app.get("/encrypt/file")
async def vault():
    html_content = f"""
        <html>
            <head>
                {BOOTSTRAP}
            </head>
            <body>
                <div class="container">
                    <div class="jumbotron jumbotron-fluid">
                      <div class="container">
                        <h1 class="display-4">Encrypt File</h1>
                        <p class="lead">After encryption password and secret file are removed.</p>
                      </div>
                    </div>
                    <hr>
                    <div class="form-group mb-4">
                        <div class="d-flex justify-content-center">
                            <a class="btn btn-outline-info btn-block" href="javascript:history.back()">Back</a>
                        </div>
                    </div>
                    <form action="/encrypt/file" accept-charset="UTF-8" method="post">
                        <div class="form-group mb-2">
                            <label>Password</label>
                            <input name="password" type="password" class="form-control" placeholder="************">
                        </div>                
                        <div class="form-group mb-2">
                            <label>Secret</label>
                            <textarea name="secret" class="form-control" rows="20"></textarea>
                        </div>
                        <div class="form-group mb-2">
                            <label>Vault ID</label>
                            <input name="vault_id" type="text" class="form-control">
                        </div> 
                        <div class="form-group">
                            <div class="d-grid">
                              <button class="btn btn-outline-danger" type="submit">Encrypt</button>
                            </div>
                        </div>
                    </form>
                </div>
            </body>
        </html>
        """
    return HTMLResponse(content=html_content, status_code=200)


@app.get("/encrypt/string")
async def vault():
    html_content = f"""
        <html>
            <head>
                {BOOTSTRAP}
            </head>
            <body>
                <div class="container">
                    <div class="jumbotron jumbotron-fluid">
                      <div class="container">
                        <h1 class="display-4">Encrypt String</h1>
                        <p class="lead">After encryption password is removed.</p>
                      </div>
                    </div>
                    <hr>
                    <div class="form-group mb-4">
                        <div class="d-flex justify-content-center">
                            <a class="btn btn-outline-info btn-block" href="javascript:history.back()">Back</a>
                        </div>
                    </div>
                    <form action="/encrypt/string" accept-charset="UTF-8" method="post">
                        <div class="form-group mb-2">
                            <label>Password</label>
                            <input name="password" type="password" class="form-control" placeholder="************">
                        </div>
                        <div class="form-group mb-2">
                            <label>Secret</label>
                            <input name="secret" type="text" class="form-control">
                        </div>             
                         <div class="form-group mb-2">
                            <label>Name</label>
                            <input name="name" type="text" class="form-control">
                        </div> 
                        <div class="form-group mb-2">
                            <label>Vault ID</label>
                            <input name="vault_id" type="text" class="form-control">
                        </div>  
                        <div class="form-group">
                            <div class="d-grid gap-2">
                              <button class="btn btn-outline-danger" type="submit">Encrypt</button>
                            </div>
                        </div>
                    </form>
                </div>
            </body>
        </html>
        """
    return HTMLResponse(content=html_content, status_code=200)


class AnsibleEncrypt:
    def __init__(self, password, secret, vault_id=None, name=None):
        self.password = self.write_file(password)

        if name:
            self.secret = secret
        else:
            self.secret = self.write_file(secret)

        self.output = self.write_file("")
        self.vault_id = vault_id
        self.name = name

    def get_random(self):
        return str(uuid.uuid4())

    def write_file(self, content):
        file = open(self.get_random(), "w")
        file.write(content)
        file.close()

        return file.name

    def delete_file(self, file_names):
        for file in file_names:
            if os.path.isfile(file):
                os.remove(file)

    def encrypt(self):
        if self.name and self.vault_id is None:
            command = f"ansible-vault encrypt_string --vault-password-file {self.password} --name {self.name} {self.secret} > {self.output} "
            subprocess.call(command, shell=True)
            encrypted = open(self.output, 'r').read()
        elif self.name and self.vault_id:
            command = f"ansible-vault encrypt_string --vault-id {self.vault_id}@{self.password} --name {self.name} {self.secret} > {self.output} "
            subprocess.call(command, shell=True)
            encrypted = open(self.output, 'r').read()
        elif self.vault_id:
            command = f"ansible-vault encrypt --vault-id {self.vault_id}@{self.password} {self.secret}"
            subprocess.call(command, shell=True)
            encrypted = open(self.secret, 'r').read()
        else:
            command = f"ansible-vault encrypt {self.secret} --vault-password-file {self.password}"
            subprocess.call(command, shell=True)
            encrypted = open(self.secret, 'r').read()

        self.delete_file([self.password, self.secret, self.output])

        return encrypted.replace("\n", "<br />")


@app.post("/encrypt/file")
async def encrypt_file(secret: str = Form(...), password: str = Form(...),
                       vault_id: Optional[str] = Form(None)):
    vault = AnsibleEncrypt(password, secret, vault_id)

    return HTMLResponse(content=vault.encrypt(), status_code=200)


@app.post("/encrypt/string")
async def encrypt_string(password: str = Form(...), secret: str = Form(...),
                         vault_id: Optional[str] = Form(None), name: str = Form(...)):
    vault = AnsibleEncrypt(password, secret, vault_id, name)

    return HTMLResponse(content=vault.encrypt(), status_code=200)

```
