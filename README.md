# PyLearn

PyLearn este o platformă în limba română pentru învățarea programării în Python. Elevii pot porni de la un exemplu de cod dintr-o lecție, îl pot modifica în browser și pot trece apoi la problemele asociate. Profesorii pot grupa problemele în teme și pot vedea soluțiile trimise de fiecare elev.

[Deschide PyLearn](https://pylearn.ro/)

## Ce poți face pe platformă

- **Învață pe capitole.** Lecțiile sunt organizate pe clase, secțiuni și capitole. Ele combină explicații în Markdown, formule matematice și exemple Python executabile.
- **Rezolvă probleme.** Caută după titlu sau filtrează după capitol, dificultate și stadiul rezolvării. În editor poți rula codul pe un exemplu înainte să trimiți soluția pentru evaluare automată.
- **Urmărește-ți progresul.** Profilul reunește istoricul soluțiilor, punctele XP și calendarul de activitate. Clasamentul arată rezultatele celorlalți utilizatori.
- **Lucrează cu o clasă.** Profesorul creează clasa și distribuie codul de înscriere. Apoi poate publica anunțuri, stabili termene pentru teme și consulta codul trimis de elevi pentru fiecare problemă.

## Cum este construit proiectul

| Componentă | Rol |
| --- | --- |
| [Interfața React](src/) | Vite, React Router și Tailwind CSS. Editorul de cod folosește Monaco pe desktop și CodeMirror pe mobil. |
| [API-ul de evaluare](backend/code-runner-api/) | Un Cloudflare Worker scris în TypeScript trimite codul Python către Judge0. `POST /run` rulează un exemplu, iar `POST /submit` evaluează soluția unui utilizator autentificat pe testele problemei și înregistrează rezultatul. |
| [Baza de date](supabase/) | Supabase gestionează autentificarea și datele din PostgreSQL, inclusiv lecțiile, problemele, clasele și soluțiile. Directorul conține migrări SQL, teste și exemple de conținut. |

## Pornire locală

Configurația folosită mai jos presupune Node.js 24, npm și acces la un proiect Supabase cu schema PyLearn. [Migrările](supabase/migrations/) extind o bază existentă. Pentru o instalare de la zero ai nevoie și de schema inițială, care include tabele precum `classrooms` și `classroom_members`.

### 1. Configurează interfața

Din rădăcina repository-ului, rulează `npm ci`. Creează fișierul `.env.local` și completează valorile pentru proiectul tău Supabase. Adresa API din exemplu corespunde Worker-ului pornit local.

```dotenv
VITE_SUPABASE_URL=https://<project-ref>.supabase.co
VITE_SUPABASE_ANON_KEY=<cheia-publica-anon>
VITE_API_URL=http://localhost:8787
```

### 2. Pornește API-ul de evaluare

Creează fișierul `backend/code-runner-api/.dev.vars` cu valorile aceluiași proiect Supabase. Cheia `service_role` aparține exclusiv Worker-ului.

```dotenv
SUPABASE_URL=https://<project-ref>.supabase.co
SUPABASE_SERVICE_ROLE_KEY=<cheia-secreta-service-role>
```

Într-un terminal deschis la rădăcina repository-ului, rulează următoarele comenzi.

```sh
cd backend/code-runner-api
npm ci
npm run dev
```

Poți verifica dacă Worker-ul a încărcat configurarea la [localhost:8787/health](http://localhost:8787/health). Rularea codului Python necesită acces la serviciul extern Judge0.

### 3. Pornește interfața

Într-un al doilea terminal, din rădăcina repository-ului, pornește Vite pe portul 5173.

```sh
npm run dev -- --port 5173 --strictPort
```

Deschide [localhost:5173](http://localhost:5173). API-ul acceptă cereri din browser de la această adresă.

## Verificări

Rulează comenzile de mai jos din rădăcina repository-ului.

| Comandă | Ce verifică |
| --- | --- |
| `npm run lint` | Analizează codul cu Oxlint. |
| `npm run test:activity` | Testează calendarul de activitate și calculul seriilor de zile consecutive, inclusiv la schimbarea orei în București. |
| `npm run build` | Generează versiunea de producție a interfeței în `dist/`. |
