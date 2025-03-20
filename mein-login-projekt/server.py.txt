from flask import Flask, render_template, request, redirect, url_for, session
import gspread
from oauth2client.service_account import ServiceAccountCredentials
import pandas as pd

app = Flask(__name__)
app.secret_key = "geheime_schluessel"

# Benutzerverwaltung (Einfache Liste)
users = {
    "admin": "passwort123",
    "user1": "geheim"
}

# Google Sheets API einrichten
scope = ["https://spreadsheets.google.com/feeds", "https://www.googleapis.com/auth/drive"]
creds = ServiceAccountCredentials.from_json_keyfile_name("service_account.json", scope)
client = gspread.authorize(creds)

# Google Sheets Datei öffnen
spreadsheet = client.open("DeinSheetName").sheet1  # Dein Sheet-Name


@app.route("/", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form["username"]
        password = request.form["password"]
        
        if username in users and users[username] == password:
            session["user"] = username
            return redirect(url_for("dashboard"))
        else:
            return render_template("login.html", error="Falscher Benutzername oder Passwort")
    
    return render_template("login.html")


@app.route("/dashboard")
def dashboard():
    if "user" not in session:
        return redirect(url_for("login"))

    data = spreadsheet.get_all_records()
    df = pd.DataFrame(data)
    table_html = df.to_html(classes="table table-bordered", index=False)

    return render_template("dashboard.html", table=table_html)


@app.route("/logout")
def logout():
    session.pop("user", None)
    return redirect(url_for("login"))


if __name__ == "__main__":
    app.run(debug=True)
