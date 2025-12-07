# 🏆 Google Sheets Leaderboard & Accounts Setup Gids

Volg deze stappen om je leaderboard EN accounts te koppelen aan Google Sheets.

---

## Stap 1: Maak een Google Sheet

1. Ga naar [Google Sheets](https://sheets.google.com)
2. Klik op **+ Blank** om een nieuwe sheet te maken
3. Noem het "4ward Database"
4. Maak de volgende kolommen in rij 1:
   - **A1**: `name`
   - **B1**: `password`
   - **C1**: `stars`
   - **D1**: `date`

---

## Stap 2: Maak een Google Apps Script

1. Klik bovenaan op **Extensies** → **Apps Script**
2. Verwijder alle code en plak dit:

```javascript
const SHEET_NAME = 'Blad1'; // Pas aan als je sheet anders heet

function doGet(e) {
  const action = e.parameter.action;
  
  if (action === 'get') {
    return getLeaderboard();
  }
  
  if (action === 'checkAccount') {
    return checkAccountExists(e.parameter.name);
  }
  
  if (action === 'login') {
    return loginUser(e.parameter.name, e.parameter.password);
  }
  
  return ContentService.createTextOutput(JSON.stringify({ error: 'Unknown action' }))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    
    if (data.action === 'add') {
      return addToLeaderboard(data.name, data.stars, data.date);
    }
    
    if (data.action === 'register') {
      return registerUser(data.name, data.password, data.stars, data.date);
    }
    
    return ContentService.createTextOutput(JSON.stringify({ error: 'Unknown action' }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

// Check of een account bestaat
function checkAccountExists(name) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] && data[i][0].toString().toLowerCase() === name.toLowerCase()) {
      return ContentService.createTextOutput(JSON.stringify({ exists: true }))
        .setMimeType(ContentService.MimeType.JSON);
    }
  }
  
  return ContentService.createTextOutput(JSON.stringify({ exists: false }))
    .setMimeType(ContentService.MimeType.JSON);
}

// Registreer een nieuwe gebruiker
function registerUser(name, password, stars, date) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  // Check of naam al bestaat
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] && data[i][0].toString().toLowerCase() === name.toLowerCase()) {
      return ContentService.createTextOutput(JSON.stringify({ success: false, error: 'Naam al in gebruik' }))
        .setMimeType(ContentService.MimeType.JSON);
    }
  }
  
  // Nieuwe gebruiker toevoegen
  sheet.appendRow([name, password, stars || 0, date]);
  
  return ContentService.createTextOutput(JSON.stringify({ success: true }))
    .setMimeType(ContentService.MimeType.JSON);
}

// Login gebruiker
function loginUser(name, password) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] && data[i][0].toString().toLowerCase() === name.toLowerCase()) {
      // Account gevonden, check wachtwoord
      if (data[i][1] === password) {
        return ContentService.createTextOutput(JSON.stringify({ 
          success: true, 
          name: data[i][0],
          stars: parseInt(data[i][2]) || 0
        }))
          .setMimeType(ContentService.MimeType.JSON);
      } else {
        return ContentService.createTextOutput(JSON.stringify({ 
          success: false, 
          error: 'Onjuist emoji wachtwoord' 
        }))
          .setMimeType(ContentService.MimeType.JSON);
      }
    }
  }
  
  return ContentService.createTextOutput(JSON.stringify({ 
    success: false, 
    error: 'Account niet gevonden. Registreer eerst!' 
  }))
    .setMimeType(ContentService.MimeType.JSON);
}

// Haal leaderboard op (zonder wachtwoorden!)
function getLeaderboard() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  const leaderboard = [];
  for (let i = 1; i < data.length; i++) {
    if (data[i][0]) {
      leaderboard.push({
        name: data[i][0],
        stars: parseInt(data[i][2]) || 0, // Kolom C is nu stars
        date: data[i][3] || ''
      });
    }
  }
  
  leaderboard.sort((a, b) => b.stars - a.stars);
  
  return ContentService.createTextOutput(JSON.stringify({ leaderboard }))
    .setMimeType(ContentService.MimeType.JSON);
}

// Update leaderboard (voor sterren updates)
function addToLeaderboard(name, stars, date) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] && data[i][0].toString().toLowerCase() === name.toLowerCase()) {
      // Update als nieuwe score hoger is
      if (stars > (parseInt(data[i][2]) || 0)) {
        sheet.getRange(i + 1, 3).setValue(stars); // Kolom C
        sheet.getRange(i + 1, 4).setValue(date);   // Kolom D
      }
      return ContentService.createTextOutput(JSON.stringify({ success: true, updated: true }))
        .setMimeType(ContentService.MimeType.JSON);
    }
  }
  
  return ContentService.createTextOutput(JSON.stringify({ success: false, error: 'User not found' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Klik op **Opslaan** (💾 icoon of Ctrl+S)
4. Geef het project een naam zoals "4ward Database Script"

---

## Stap 3: Deploy als Web App

1. Klik op **Deploy** → **New deployment**
2. Klik op het tandwiel ⚙️ → **Web app**
3. Vul in:
   - **Description**: 4ward API
   - **Execute as**: Me
   - **Who has access**: **Anyone** (belangrijk!)
4. Klik op **Deploy**
5. Je krijgt een **URL** - kopieer deze! Het ziet er zo uit:
   ```
   https://script.google.com/macros/s/AKfycbx.../exec
   ```

---

## Stap 4: Voeg de URL toe aan je website

1. Open `index.html`
2. Zoek naar deze regel:
   ```javascript
   GOOGLE_SCRIPT_URL: 'JOUW_GOOGLE_SCRIPT_URL_HIER',
   ```
3. Vervang `JOUW_GOOGLE_SCRIPT_URL_HIER` met je gekopieerde URL

---

## ⚠️ BELANGRIJK: Script opnieuw deployen

Als je al een oude versie had, moet je een **nieuwe deployment** maken:

1. Ga naar Apps Script
2. Klik op **Deploy** → **New deployment** (NIET "Manage deployments")
3. Maak een nieuwe Web app deployment
4. Kopieer de **NIEUWE URL** en vervang de oude in je index.html

---

## 🎉 Klaar!

Nu werkt het zo:
- ✅ Gebruikers moeten registreren met naam + 3 emoji's als wachtwoord
- ✅ Dubbele namen zijn niet toegestaan
- ✅ Bij inloggen wordt het emoji wachtwoord gecontroleerd
- ✅ Sterren worden veilig opgeslagen in Google Sheets
- ✅ Je kunt in de Google Sheet:
  - **Accounts verwijderen**: Verwijder de rij
  - **Sterren aanpassen**: Verander het getal in kolom C
  - **Wachtwoord resetten**: Pas de emoji's aan in kolom B

---

## 🔐 Beveiliging

De Google Sheet bevat:
| name | password | stars | date |
|------|----------|-------|------|
| Jan | 😀🐶⭐ | 15 | 5/12/2024 |
| Lisa | 🎮🚀🌈 | 23 | 5/12/2024 |

**Let op:** Het wachtwoord (emoji's) is zichtbaar in de sheet. Dit is geen high-security systeem, maar voorkomt dat kinderen zomaar kunnen valsspelen.

---

## ❓ Troubleshooting

**Het werkt niet?**
1. Check of de URL correct is gekopieerd
2. Zorg dat "Who has access" op "Anyone" staat
3. Maak een NIEUWE deployment (niet manage existing)
3. Als je het script aanpast, moet je opnieuw deployen:
   - **Deploy** → **Manage deployments** → **Edit** (✏️) → **New version** → **Deploy**

**Leaderboard laadt niet?**
- Het kan zijn dat de eerste keer laden even duurt (cold start)
- Check de browser console (F12) voor foutmeldingen

**Sheet naam is anders?**
- Als je sheet niet "Blad1" heet, pas dan `const SHEET_NAME = 'Blad1';` aan in het script
