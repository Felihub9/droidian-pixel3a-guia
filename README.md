# Instal·lació de Droidian a un Google Pixel 3a (sargo) venint d'Android 11

Aquesta guia documenta, pas a pas, el procés real que vaig seguir per instal·lar
[Droidian](https://droidian.org) (una distribució Linux mòbil basada en Debian) en un
Google Pixel 3a.

## Motivació i context

Droidian és Debian de veritat al mòbil, sense Google ni telemetria, amb terminal, SSH i apt.
A més, dona una segona vida als telèfons que ja no reben actualitzacions oficials.
No és el més còmode per a l’ús diari, però és ideal per aprendre, trastejar i fer-lo
servir com a petit servidor o dispositiu domèstic.

El Pixel 3a que havia trobat portava **Android 11**, i havia llegit que Droidian pel 
Pixel 3a es basa en una imatge vendor **d'Android 9**. No sabia si això seria un problema 
bloquejant, si calia baixar de versió primer, o si era arriscat fer-ho.

Vaig investigar-ho abans de comprar i vaig confirmar:

- El Pixel 3a (`sargo`) **no té protecció anti-rollback** al bootloader (a diferència
  dels Pixel 6 i posteriors), així que baixar de versió és segur.
- Cal baixar a Android 9 **abans** de flashejar Droidian, ja que el port es basa en
  aquesta versió del vendor partition.
- **Punt crític a comprovar abans de comprar:** que el telèfon no sigui un model
  bloquejat d'operadora (per exemple, Verizon als EUA), ja que en aquests models
  l'opció "OEM unlocking" no es pot activar mai, fent impossible tot el procés.

Aquest document recull el procés complet ja fet i verificat en un dispositiu real.

## ⚠️ Avís important

- Tot el procés **esborra completament les dades del telèfon**. Fes còpia de seguretat
  abans si cal.
- Desbloquejar el bootloader **anul·la la garantia** (si en tenies).
- Segueix els passos amb calma i llegeix cada comanda abans d'executar-la.
- Aquest procés es va fer en un Pixel 3a concret; en altres unitats pot variar
  lleugerament.

## Requisits previs

- Un Google Pixel 3a (`sargo`) o 3a XL (`bonito`) **no bloquejat per operadora**.
- Un PC amb Linux (aquí es va fer servir Debian amb XFCE i KDE).
- Paquets `adb` i `fastboot` instal·lats (`sudo apt install adb fastboot`).
- Cable USB en bon estat.
- Connexió a internet per descarregar les imatges (uns quants GB en total).

## Pas 1 — Comprovar que el bootloader es pot desbloquejar

Al telèfon, amb Android arrencat normalment:

1. **Ajustos → Sistema → Quant al telèfon**
2. Toca 7 vegades seguides sobre "Número de compilació" fins que aparegui
   "Ja ets desenvolupador!"
3. Torna enrere → **Sistema → Opcions de desenvolupador**
4. Activa **"Depuració USB"**
5. Comprova que **"Desbloqueig d'OEM"** es pot activar (si surt en gris i no es pot
   tocar, molt probablement el telèfon és d'un model bloquejat i el procés no es pot
   continuar)
6. Activa "Desbloqueig d'OEM"

## Pas 2 — Connectar el telèfon i verificar la comunicació ADB

Amb el telèfon engegat i connectat per USB:

```bash
adb devices
```

Accepta el diàleg d'autorització que apareix al telèfon. Torna a executar la comanda
fins que l'estat sigui `device` (no `unauthorized`):

```bash
adb devices
# hauria de mostrar:
# <número de sèrie>    device
```

## Pas 3 — Entrar en mode Fastboot i desbloquejar el bootloader

```bash
adb reboot bootloader
```

Un cop el telèfon mostri la pantalla "Fastboot Mode", comprova que el PC el detecta:

```bash
fastboot devices
```

Desbloqueja el bootloader (**això esborrarà totes les dades**):

```bash
fastboot flashing unlock
```

Confirma l'acció amb els botons físics de volum/encendre al telèfon. Verifica el
resultat:

```bash
fastboot getvar unlocked
# hauria de respondre: unlocked: yes
```

## Pas 4 — Descarregar la imatge de fàbrica d'Android 9

Ves a la pàgina oficial de Google:

<https://developers.google.com/android/images>

Prem "Acknowledge", busca el dispositiu **"sargo"** i descarrega la build:

- **9.0.0 (PQ3B.190801.002, Aug 2019)**

Descomprimeix-la:

```bash
unzip sargo-pq3b.190801.002-factory-<hash>.zip
cd sargo-pq3b.190801.002
```

## Pas 5 — Baixar el telèfon a Android 9

Amb el telèfon **en mode Fastboot** i detectat (`fastboot devices`), revisa el
contingut de l'script abans d'executar-lo (és una bona pràctica general no executar
scripts sense llegir-los primer):

```bash
cat flash-all.sh
```

Si tot sembla correcte (flasheja bootloader, radio, i el sistema, en aquest ordre),
executa'l:

```bash
chmod +x flash-all.sh
./flash-all.sh
```

Aquest procés triga uns 2-3 minuts. **No desconnectis el cable ni toquis el telèfon**
fins que acabi. El telèfon es reiniciarà automàticament en Android 9 al final.

> **Nota sobre un problema conegut:** en alguns fòrums es documenten casos de
> *bootloop* en baixar d'Android 12 a Android 9 fent servir comandes de `fastboot`
> soltes manualment (flashejar bootloader, radio i sistema per separat amb pauses
> manuals). En aquest procés, fent servir l'script oficial `flash-all.sh` complet
> (que inclou els `sleep` necessaris entre passos), **no vam patir aquest problema**,
> ni baixant des d'Android 11.

Un cop arrencat en Android 9, no cal completar l'assistent de configuració inicial.
Torna a activar Opcions de desenvolupador i Depuració USB (es reinicien en cada flash,
però **el desbloqueig del bootloader es manté**, ja que és un estat de maquinari, no
una configuració d'Android):

```bash
# Repeteix el pas 1, punts 1-4 (no cal tornar a activar OEM unlocking)
```

## Pas 6 — Descarregar Droidian

Amb `curl` es pot consultar directament l'API de GitHub per obtenir els enllaços de
la darrera versió:

```bash
curl -s https://api.github.com/repos/droidian-images/droidian/releases/latest \
  | grep -E '"(name|browser_download_url)"'
```

Busca el fitxer amb el patró `droidian-OFFICIAL-phosh-phone-google_sargo-api28-*.zip`
i el fitxer `SHA256SUMS`. Descarrega'ls:

```bash
wget <url del zip de sargo>
wget <url del SHA256SUMS>
```

**Verifica la integritat de la descàrrega** abans de continuar:

```bash
grep sargo SHA256SUMS
sha256sum droidian-OFFICIAL-phosh-phone-google_sargo-api28-*.zip
```

Els dos hashos han de coincidir exactament.

## Pas 7 — Flashejar Droidian

Descomprimeix el paquet:

```bash
unzip droidian-OFFICIAL-phosh-phone-google_sargo-api28-*.zip -d droidian-sargo
cd droidian-sargo
```

Revisa el `README` i el script `flash_all.sh` inclosos (bona pràctica de seguretat
abans d'executar qualsevol script descarregat). Amb el telèfon en mode Fastboot i
detectat:

```bash
chmod +x flash_all.sh
./flash_all.sh
```

Aquest pas flasheja `boot`, `dtbo`, `vbmeta` i el sistema Droidian complet
(`userdata.img`, uns 3-4 GB), i triga uns 2-3 minuts. En acabar hauria de mostrar:

```
I: Flashing completed
```

El telèfon es reiniciarà sol cap a Droidian.

## Resultat

Després d'aquest procés, el telèfon arrenca amb **Droidian 101** i l'entorn
d'escriptori **Phosh**. La contrasenya de desbloqueig per defecte sol ser `1234`
(es recomana canviar-la).

## Reconeixement

Aquest procés l'he fet amb l'ajuda de Claude (Anthropic), tant per investigar prèviament
si el dispositiu i la versió d'Android eren compatibles amb Droidian abans d'obtenir-lo,
com per guiar-me pas a pas durant tota la instal·lació. Totes les comandes i decisions finals
les he executat jo directament al meudispositiu.

He fet això com a exercici propi d'aprenentatge i ho publico per ajudar altres persones
que els pugui interessar fer aquest procés.

## Referències

- [Droidian — pàgina oficial](https://droidian.org)
- [Droidian images — GitHub](https://github.com/droidian-images/droidian)
- [Factory Images for Nexus and Pixel Devices — Google](https://developers.google.com/android/images)
- [Fòrum de Droidian — dispositiu Pixel 3a/3a XL](https://forum.droidian.org/t/google-pixel-3a-3a-xl/47)
