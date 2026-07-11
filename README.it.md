# Creare un Server Git Privato da Zero

Questa guida illustra i passaggi per configurare un Server Git autonomo su una macchina Linux. Anche se piattaforme cloud come GitHub sono fantastiche, creare il tuo server locale offre dei vantaggi unici:

- **Controllo Totale e Privacy**: Il tuo codice risiede solo sul tuo hardware. Non devi affidare i tuoi progetti personali o i tuoi esperimenti a servizi esterni.
- **Lavorare Offline**: Puoi salvare il codice e collaborare con altri computer sulla stessa rete locale (LAN) anche quando non c'è connessione internet.
- **Imparare "Sotto il Cofano"**: Costruire un server da zero ti fa capire davvero come funzionano gli utenti Linux, i permessi dei file e le chiavi di sicurezza SSH.
- **Nessun Limite**: Hai spazio infinito per i tuoi progetti privati, senza doverti preoccupare di abbonamenti o limiti sulle dimensioni dei file.

Questo progetto è una guida semplice e didattica pensata per aiutarti a costruire da zero la tua infrastruttura indipendente per il controllo di versione.

*Nota: Per la versione in inglese di questa guida, consulta [README.md](README.md).*

---

## Prerequisiti
Prima di iniziare, assicurati di avere:
- Una macchina Linux (fisica o virtuale) a cui hai accesso tramite terminale.
- Conoscenze di base della riga di comando (navigare tra le cartelle, eseguire comandi).
- Privilegi `sudo` sulla macchina (cioè la possibilità di eseguire comandi come amministratore).

---

## 1. Installazione di Git
Assicuriamoci che Git sia installato sul sistema. Il comando varia in base alla distribuzione Linux in uso:

**Debian/Ubuntu:**
```bash
sudo apt update
sudo apt install git
```

**CentOS/RHEL/Fedora:**
```bash
sudo dnf install git  # Usa 'yum' sui sistemi più vecchi
```

**Arch Linux:**
```bash
sudo pacman -S git
```

## 2. Configurazione dell'Utente Server (Lato Server)
Per isolare i progetti e garantire la sicurezza del sistema operativo, creiamo un utente dedicato esclusivamente alle operazioni Git. Questo utente non avrà permessi di root (`sudo`).
```bash
# Creiamo l'utente e impostiamo la password
sudo useradd git-user
sudo passwd git-user

# Creiamo la cartella "home" per questo utente e gli assegniamo la proprietà
sudo mkdir -p /home/git-user
sudo chown -R git-user:git-user /home/git-user
```

## 3. Generazione e Autenticazione delle Chiavi SSH (Lato Client)
Comprendere le chiavi SSH è fondamentale. Esse permettono una connessione sicura al server, evitando di inserire la password in chiaro a ogni push o pull.

Sul computer locale dello sviluppatore:
```bash
# Genera la coppia di chiavi crittografiche (premi INVIO per le impostazioni predefinite)
ssh-keygen -t rsa -C "developer@example.com"

# Invia la chiave pubblica (la "serratura") al server
cat ~/.ssh/id_rsa.pub | ssh git-user@localhost 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```
*(Durante questa operazione verrà chiesta la password di `git-user` un'ultima volta per autorizzare il trasferimento).*

## 4. Creazione del Repository Bare (Lato Server)
Un repository sul server deve essere di tipo "bare" (nudo). Agisce come database centrale: riceve i dati ma non possiede file visibili e modificabili direttamente sul server.
```bash
# Cambiamo utente accedendo come l'amministratore del server Git
su git-user

# Creiamo la cartella e inizializziamo il repository bare
mkdir -p /home/git-user/mio-progetto.git
cd /home/git-user/mio-progetto.git
git init --bare
```

## 5. Impostazione del Repository Locale (Lato Client)
Torniamo ora nei panni dello sviluppatore per creare il progetto locale sul nostro computer.
```bash
# Usciamo dall'utente server per tornare al nostro utente standard
exit

# Creiamo la cartella di lavoro sul nostro PC
mkdir -p /home/tuoutente/progetti/mio-progetto
cd /home/tuoutente/progetti/mio-progetto

# Inizializziamo Git e configuriamo la firma dell'autore
git init
git config --global user.name "Il Tuo Nome"
git config --global user.email "developer@example.com"

# Creiamo un file iniziale per testare il salvataggio
echo "# Progetto di Test" > readme.md
```

## 6. Commit e Push verso il Server Privato
Prepariamo il codice e inviamolo al server privato appena configurato per vedere il sistema in azione.
```bash
# Aggiungiamo i file alla staging area
git add .
git commit -m "Commit iniziale del progetto"

# Configuriamo il "remote" per puntare al nostro server privato
git remote add origin git-user@localhost:/home/git-user/mio-progetto.git

# Eseguiamo il push del codice sul server
# Nota: a seconda della versione di Git, il branch predefinito potrebbe chiamarsi 'main' invece di 'master'.
# Se il comando qui sotto fallisce, prova con: git push origin main
git push origin master
```

## 7. Verifica Finale dell'Architettura
Per confermare che il server stia effettivamente funzionando come hub centrale, simuliamo l'arrivo di un nuovo sviluppatore clonando il progetto da zero in un'altra cartella:
```bash
cd /home/tuoutente/progetti
git clone git-user@localhost:/home/git-user/mio-progetto.git
```
Se l'operazione di `clone` ha successo e la cartella scaricata contiene il file `readme.md`, l'architettura didattica del Server Git è stata compresa e configurata con successo!

---

## 8. Collegamenti Utili e Risorse
Per approfondire i concetti esplorati in questa guida, ecco alcuni link ufficiali utili:
- [Libro Ufficiale "Pro Git" (Italiano)](https://git-scm.com/book/it/v2)
- [Git sul Server - Configurare il Server (Guida Ufficiale)](https://git-scm.com/book/it/v2/Git-sul-Server-Configurare-il-Server)
- [Documentazione OpenSSH](https://www.openssh.com/manual.html)
- [Guida ai Permessi File su Linux](https://wiki.ubuntu-it.org/AmministrazioneSistema/PermessiFile)
