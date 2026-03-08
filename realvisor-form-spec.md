# Specifikace kontaktního formuláře pro Realvisor API

## Formulář na stránce Fix&Flip.cz

### Umístění
- Sekce `#tip` (řádek 1476-1538 v index.html)
- Formulář ID: `tipForm`

### Struktura polí

```html
<form id="tipForm" onsubmit="handleFormSubmit(event)">
```

#### 1. Jméno (povinné)
- **ID**: `name`
- **Name**: `name`
- **Type**: `text`
- **Placeholder**: "Jan Novák"
- **Required**: ano

#### 2. Telefon (povinné)
- **ID**: `phone`
- **Name**: `phone`
- **Type**: `tel`
- **Placeholder**: "+420 xxx xxx xxx"
- **Required**: ano

#### 3. E-mail (nepovinné)
- **ID**: `email`
- **Name**: `email`
- **Type**: `email`
- **Placeholder**: "jan@email.cz"
- **Required**: ne

#### 4. Typ nemovitosti (nepovinné)
- **ID**: `propertyType`
- **Name**: `propertyType`
- **Type**: `select`
- **Možnosti**:
  - `""` - "Vyberte..."
  - `"byt-panel"` - "Byt v panelovém domě"
  - `"byt-cihla"` - "Byt v cihlovém domě"
  - `"dum"` - "Rodinný dům"
  - `"jine"` - "Jiné"

#### 5. Lokalita / adresa (povinné)
- **ID**: `location`
- **Name**: `location`
- **Type**: `text`
- **Placeholder**: "Např. Praha 3 - Vinohrady, ulice..."
- **Required**: ano

#### 6. Popis / poznámka (nepovinné)
- **ID**: `note`
- **Name**: `note`
- **Type**: `textarea`
- **Placeholder**: "Cokoliv víte o nemovitosti - stav, cena, kontakt na majitele..."
- **Required**: ne

---

## Data k odeslání do Realvisor API

### JSON struktura
```json
{
  "name": "Jan Novák",
  "phone": "+420 123 456 789",
  "email": "jan@email.cz",
  "propertyType": "byt-panel",
  "location": "Praha 3 - Vinohrady, Korunní 123",
  "note": "Byt po babičce, původní stav, majitel chce rychle prodat"
}
```

### Mapování polí
| HTML pole | API parametr | Typ | Povinné |
|-----------|--------------|-----|---------|
| `name` | `name` | string | ano |
| `phone` | `phone` | string | ano |
| `email` | `email` | string | ne |
| `propertyType` | `propertyType` | string | ne |
| `location` | `location` | string | ano |
| `note` | `note` | string | ne |

---

## Aktuální implementace (simulovaná)

Formulář má aktuálně mock implementaci na řádcích 1684-1707:

```javascript
function handleFormSubmit(e) {
    e.preventDefault();
    const form = e.target;
    const btn = form.querySelector('button[type="submit"]');
    const originalText = btn.innerHTML;

    btn.innerHTML = 'Odesílám...';
    btn.disabled = true;

    // Simulate send - replace with actual endpoint
    setTimeout(() => {
        btn.innerHTML = 'Odesláno! Ozveme se vám.';
        btn.style.background = '#22c55e';
        btn.style.boxShadow = '0 4px 16px rgba(34,197,94,0.3)';

        setTimeout(() => {
            form.reset();
            btn.innerHTML = originalText;
            btn.disabled = false;
            btn.style.background = '';
            btn.style.boxShadow = '';
        }, 4000);
    }, 1000);
}
```

---

## Co potřebuji od Realvisor

1. **API endpoint URL** - kam posílat POST request
2. **Autentizace** - API klíč, token, nebo jiný způsob autentizace
3. **Formát požadavku** - JSON, FormData, nebo jiný formát
4. **Požadované hlavičky** - Content-Type, Authorization, atd.
5. **Struktura odpovědi** - jak vypadá success/error response
6. **Rate limiting** - jsou nějaká omezení na počet requestů?
7. **CORS nastavení** - je potřeba řešit CORS pro frontend volání?

---

## Příklad implementace s fetch API

```javascript
async function handleFormSubmit(e) {
    e.preventDefault();
    const form = e.target;
    const btn = form.querySelector('button[type="submit"]');
    const originalText = btn.innerHTML;

    // Collect form data
    const formData = {
        name: form.name.value,
        phone: form.phone.value,
        email: form.email.value || null,
        propertyType: form.propertyType.value || null,
        location: form.location.value,
        note: form.note.value || null
    };

    btn.innerHTML = 'Odesílám...';
    btn.disabled = true;

    try {
        const response = await fetch('REALVISOR_API_ENDPOINT', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer YOUR_API_KEY' // pokud je potřeba
            },
            body: JSON.stringify(formData)
        });

        if (!response.ok) {
            throw new Error('Chyba při odesílání');
        }

        const result = await response.json();

        // Success
        btn.innerHTML = 'Odesláno! Ozveme se vám.';
        btn.style.background = '#22c55e';
        btn.style.boxShadow = '0 4px 16px rgba(34,197,94,0.3)';

        setTimeout(() => {
            form.reset();
            btn.innerHTML = originalText;
            btn.disabled = false;
            btn.style.background = '';
            btn.style.boxShadow = '';
        }, 4000);

    } catch (error) {
        console.error('Error:', error);
        btn.innerHTML = 'Chyba při odesílání. Zkuste znovu.';
        btn.style.background = '#ef4444';
        
        setTimeout(() => {
            btn.innerHTML = originalText;
            btn.disabled = false;
            btn.style.background = '';
        }, 3000);
    }
}
```

---

## Poznámky

- Formulář je umístěn v tmavé sekci s glassmorphism efektem
- Po úspěšném odeslání se formulář resetuje po 4 sekundách
- Tlačítko má vizuální feedback (zelená barva pro success, červená pro error)
- Všechna povinná pole jsou označena atributem `required`
- Telefon a email mají správné HTML5 typy pro lepší UX na mobilech
