# Frontoffice

**Frontoffice-ul** reprezintă aplicația sau portalul public prin care **solicitantul** accesează serviciul pentru a depune o solicitare. Acesta este principalul punct de contact între cetățean (sau persoană juridică) și prestatorul de servicii.

## Funcționalități principale

Frontoffice-ul trebuie să ofere o experiență clară și simplificată pentru utilizatorul final. Printre funcționalitățile esențiale se regăsesc:

* **Depunerea solicitărilor** – formularul prin care utilizatorul introduce datele necesare pentru inițierea serviciului;
* **Verificarea statutului solicitării** – urmărirea stadiului în care se află cererea (ex: „în analiză”, „finalizată”, „respinse” etc.);
* **Verificarea documentului eliberat** – în cazul în care serviciul generează un document oficial, utilizatorul poate verifica validitatea acestuia (ex: certificat digital, autorizație etc.);
* **Accesarea termenilor și condițiilor** – informarea solicitantului cu privire la condițiile de utilizare a serviciului.

## Considerente suplimentare

* Dacă serviciul **impune autentificare** (ex: prin MPass), aplicația Frontoffice trebuie să integreze mecanismele de autentificare relevante;
* Dacă serviciul permite **mai multe tipuri de solicitări**, poate fi utilă o **pagină de start** care să ghideze utilizatorul în alegerea formularului potrivit;
* În cazul în care se dorește integrarea într-un portal existent (ex: **Portalul Serviciilor Publice**), este posibil ca Frontoffice-ul să se rezume doar la implementarea funcționalității de preluare și procesare a solicitărilor, în timp ce descrierea serviciului este afișată direct în portalul public.

## Exemplu de arhitectură

```text
Utilizator final
     ↓
[ Frontoffice ]
     ↓
[ Backoffice / Sistem de procesare ]
```

> ℹ️ **Notă:** Deși în unele diagrame apare \[ API intermediar (Clients API) ] între Frontoffice și Backoffice, acest API **nu este utilizat direct de către Frontoffice**, ci este destinat pentru integrarea cu **sisteme externe** care au nevoie să consume serviciul fără interfața grafică Frontoffice.

> 🛠️ **Notă:** Frontoffice-ul trebuie gândit din perspectiva utilizatorului final. Accesibilitatea, claritatea formularelor și simplificarea interacțiunilor sunt esențiale pentru creșterea gradului de utilizare a serviciului.

---

Frontoffice-ul este doar unul dintre componentele unei soluții FOD, dar joacă un rol esențial în succesul adoptării digitale a serviciilor publice.

---

## Depunerea unei solicitări

Depunerea unei solicitări este cea mai utilizată funcționalitate a Frontoffice-ului și constă în **colectarea datelor de la solicitant** necesare prestării unui anumit serviciu.

> ⚠️ Este esențial ca datele colectate să fie **doar cele strict necesare**. Nu se recomandă colectarea de informații ce pot fi obținute din alte surse oficiale (ex: datele cu caracter personal disponibile dezvoltatorului prin autentificare sau integrare cu registre naționale).

Pe baza experienței acumulate în dezvoltarea mai multor servicii, au fost identificate **mai multe etape** prin care trebuie să treacă solicitantul în cadrul procesului de depunere. Pentru fiecare etapă, au fost dezvoltate **componente specializate**, grupate în cadrul unui **comportament de tip Wizard** (asistent pas-cu-pas).

Această structurare are impact atât la nivel **funcțional**, cât și **tehnic**, deoarece impune:

* Un flux clar de colectare a datelor;
* Un mecanism de validare progresivă;
* Afișarea clară a fiecărei etape pentru utilizator.

### Exemple de etape posibile în procesul de depunere:

* Identificarea solicitantului (opțional, dacă este necesară autentificarea);
* Alegerea serviciului sau sub-tipului de solicitare;
* Completarea formularului de date;
* Încărcarea documentelor justificative;
* Confirmarea și transmiterea solicitării.

Fiecare pas este reprezentat printr-o secțiune distinctă în cadrul componentei Wizard, ceea ce oferă o experiență structurată și ușor de urmărit pentru utilizator.

> 💡 În cadrul serviciilor deja implementate, această funcționalitate este una dintre cele mai critice și vizibile, iar adaptarea fluxului la nevoile reale ale serviciului este esențială pentru succesul implementării.
