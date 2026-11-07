# scarica-youtube

`scarica-youtube` è uno script Bash che mostra il titolo trovato da `yt-dlp` prima del download. Premi Invio per usare il titolo originale oppure digita il nome che vuoi dare al file. Lo script aggiunge l’estensione del formato scaricato.

## Dipendenze

- **Bash 4.4 o successivo** e **`yt-dlp`** disponibili nel `PATH`.
- **`ffmpeg` e `ffprobe`** per unire audio e video quando sono separati. L’esempio locale usa `ffmpeg` anche per generare il video di prova.
- **Un runtime JavaScript supportato e i componenti EJS** per il supporto completo a YouTube. Gli eseguibili ufficiali di `yt-dlp` includono i componenti EJS. Consulta la [guida di yt-dlp](https://github.com/yt-dlp/yt-dlp/wiki/EJS) per configurare il runtime.

`aria2c` è facoltativo e serve solo se lo selezioni come downloader esterno nelle opzioni o nella configurazione di `yt-dlp`. Consulta le [istruzioni di installazione di yt-dlp](https://github.com/yt-dlp/yt-dlp/wiki/Installation) per la tua piattaforma.

Lo script è stato verificato su Linux. Su macOS richiede una versione di Bash che soddisfi il requisito sopra. Su Windows puoi eseguirlo da un ambiente WSL con le dipendenze installate al suo interno. macOS e WSL non sono stati provati per questa versione.

## Download in parallelo

Lo script chiama `yt-dlp` prima per leggere il titolo e poi per scaricare, in sequenza. Durante il trasferimento il downloader può fare più richieste in parallelo. Il meccanismo dipende dal formato e dal downloader selezionato.

- Con il downloader nativo di `yt-dlp`, `-N` (`--concurrent-fragments`) imposta quanti segmenti HLS o DASH, già forniti dal server, scaricare contemporaneamente. [Opzioni di yt-dlp](https://github.com/yt-dlp/yt-dlp#download-options)
- Con `aria2c`, più connessioni possono scaricare intervalli diversi dello stesso file HTTP. Il server deve supportare le richieste per intervalli e il file deve essere abbastanza grande da poter essere suddiviso. [Manuale di aria2c](https://aria2.github.io/manual/en/html/aria2c.html#cmdoption-s)

Le parti vengono ricomposte nel file finale. Il guadagno dipende dalla connessione, dal formato e dai limiti del server.

## Riprendere un download interrotto

Dopo un’interruzione, anche con `Ctrl+C`, rilancia il comando con lo stesso URL, lo stesso nome di destinazione, la stessa cartella e le stesse opzioni di formato. `yt-dlp` abilita la ripresa per impostazione predefinita con `--continue`, quando la sorgente e il downloader la supportano. [Opzioni di yt-dlp](https://github.com/yt-dlp/yt-dlp#filesystem-options)

Conserva il file parziale `.part` e, se usi `aria2c`, anche il file di controllo `.aria2` che registra le parti già scaricate. [Manuale di aria2c](https://aria2.github.io/manual/en/html/aria2c.html#control-file)

## Prova con un file locale

Dalla cartella del repository, esegui questo blocco. Crea un breve video sintetico in una cartella temporanea e usa una sottocartella separata per la copia. Al prompt, premi Invio per ottenere `originale.mp4` oppure digita un nome senza estensione.

```bash
(
    set -e
    script="$PWD/scarica-youtube"
    prova=$(mktemp -d "${TMPDIR:-/tmp}/scarica-youtube.XXXXXX")
    mkdir "$prova/output"
    ffmpeg -nostdin -hide_banner -loglevel error -f lavfi \
        -i 'color=c=blue:s=320x180:r=25' -t 2 -c:v mpeg4 "$prova/originale.mp4"

    cd "$prova/output"
    "$script" "file://$prova/originale.mp4" --ignore-config --enable-file-urls
)
```

`--enable-file-urls` abilita l’accesso al file locale per questa prova. `--ignore-config` evita che le impostazioni personali di `yt-dlp` cambino il risultato. Lo script mostra la cartella temporanea in cui si trova la copia.

Le opzioni extra vanno dopo l’URL e vengono passate a `yt-dlp`. Se avvii lo script senza argomenti, ti chiede anche l’URL.

## Usi consentiti

Usa lo script solo quando hai diritto a fare una copia e il download è consentito dai termini del servizio o dalla legge applicabile. La possibilità di vedere un video pubblicamente non equivale a un permesso di scaricarlo. Rispetta i controlli di accesso e le protezioni tecniche.

Per i video che hai caricato su YouTube, la piattaforma offre il download tramite [YouTube Studio o Google Takeout](https://support.google.com/youtube/answer/56100?hl=it). Per gli altri contenuti, consulta i [Termini di servizio di YouTube](https://www.youtube.com/t/terms?hl=it&gl=IT), in particolare la sezione «Autorizzazioni e limitazioni».

## Licenza

Lo script è distribuito con licenza [MIT](LICENSE). Le dipendenze hanno le rispettive licenze.
