From a terminal, run:

```
cd /home/camara/Aegis
source .venv/bin/activate
```

Install or refresh the project and dashboard dependencies:

```
python -m pip install -e '.[dashboard,test]'
```

Start the dashboard:

```
aegis-web
```

Leave that terminal open, then visit [http://127.0.0.1:8000/](http://127.0.0.1:8000/).

If port 8000 is already occupied, find the listener:

```
ss -ltnp '( sport = :8000 )'
```

Stop the existing AEGIS server with **Ctrl+C** in its terminal, then run `aegis-web` again. Don’t kill an unidentified process.

Useful CLI commands from another terminal:

```
cd /home/camara/Aegis
source .venv/bin/activate

aegis config
aegis doctor
aegis preflight
aegis assess --preview
```

To start and save a confirmed assessment, explicitly run:

```
aegis assess
```

For non-interactive confirmation:

```
aegis assess --yes
```

Run tests and checks with:

```
pytest -q
node --check src/aegis/static/dashboard.js
git diff --check
git diff --cached --check
```

Your local `.env` is loaded when AEGIS starts from the project directory. The `test` credentials there are for local Floci only; never use them with real AWS.