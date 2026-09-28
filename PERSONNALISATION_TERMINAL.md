# Personnalisation de mon terminal PowerShell

Ce guide permet de refaire **exactement** la même personnalisation de terminal sur un autre PC :

- l'avatar GitHub `pape-medoune` en pixel art, en couleurs ;
- la bannière **MEDOUNE** (police ANSI Shadow, dégradé violet → bleu) alignée à droite de l'avatar ;
- la signature « Mouhamedoune FALL · Full-Stack Web & Mobile » et `github.com/pape-medoune` ;
- le prompt moderne sur deux lignes (utilisateur, dossier courant, branche git, flèche verte/rouge).

---

## 1. Prérequis

| Élément | Pourquoi | Comment vérifier / installer |
| :--- | :--- | :--- |
| **Windows Terminal** | Affiche les couleurs exactes (true color) et fournit la police Cascadia Mono | Installé par défaut sur Windows 11, sinon Microsoft Store → « Windows Terminal » |
| **Police Cascadia Mono** (par défaut dans Windows Terminal) | Contient les séparateurs du prompt et l'icône de branche git | Rien à faire si la police du terminal n'a pas été changée |
| **Windows PowerShell 5.1** | Version utilisée pour ce profil | `$PSVersionTable.PSVersion` |
| **Git** (optionnel) | Affiche la branche dans le prompt quand on est dans un dépôt | `git --version`, sinon https://git-scm.com |
| **Politique d'exécution `RemoteSigned`** | Autorise PowerShell à charger le profil au démarrage | voir étape 2 |

---

## 2. Autoriser le chargement du profil

Dans PowerShell (une seule fois) :

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

Vérification : `Get-ExecutionPolicy -List` doit afficher `CurrentUser  RemoteSigned`.

---

## 3. Ouvrir (ou créer) le fichier de profil

```powershell
if (-not (Test-Path $PROFILE)) { New-Item -ItemType File -Path $PROFILE -Force }
notepad $PROFILE
```

Sur mon PC actuel, le profil se trouve ici :
`C:\Users\User\OneDrive\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`

> Astuce : ce dossier est synchronisé par OneDrive. Sur un nouveau PC connecté au même compte OneDrive,
> le profil peut déjà être présent : vérifie avec `Test-Path $PROFILE`.
> Ce guide, lui, est disponible sur GitHub : https://github.com/pape-medoune/pape-medoune/blob/master/PERSONNALISATION_TERMINAL.md

---

## 4. Coller le code

Colle **tout le bloc ci-dessous au tout début** du fichier `$PROFILE`, puis enregistre.

⚠️ **Important sur l'encodage :** le code est écrit volontairement **uniquement avec des caractères ASCII**
(les blocs, coins et symboles sont générés avec `[char]0x....`). Ne remplace pas ces codes par les vrais
symboles (`█`, `╗`, `❯`…) : PowerShell 5.1 lit les fichiers sans BOM en ANSI et les afficherait cassés.

```powershell
Clear-Host

# 0. Avatar GitHub (pape-medoune) en pixel art
# Legende : # contour | H cheveux | . peau | o yeux | $ chaine en or
function Show-Avatar {
    $e = [char]27
    $colors = @{
        '#' = '230;237;243'
        'H' = '179;79;209'
        '.' = '198;138;90'
        'o' = '255;255;255'
        '$' = '255;212;59'
    }
    $pixels = @(
        '         H     H',
        '    HH    H   H',
        '      H #######',
        '       #HHHHHHH#',
        '    HH#HHHHHHHHH#',
        '   H  #HHHHHHHHH#H',
        '      #HH..H..H.# H',
        '     H#.H..H..H.# H',
        '    H #.HHHH..HH# H',
        '     #...#oH..#o#',
        '     #.....H....#',
        '      #.........#',
        '      #.....##..#',
        '      #.........#',
        '      #.........#',
        '      #....###..#',
        '      #.........#',
        '      #........#',
        '      #$..#####',
        '      #.$.#',
        '      #..$#'
    )
    # 1. Banniere MEDOUNE (police ANSI Shadow), affichee a droite de l'avatar
    # Legende : # bloc plein | = - | < > [ ] : traits et coins de l'ombre
    $glyphs = @{
        '#' = [char]0x2588; '=' = [char]0x2550; '|' = [char]0x2551
        '<' = [char]0x2554; '>' = [char]0x2557; '[' = [char]0x255A; ']' = [char]0x255D
    }
    $letters = @(
        '###>   ###>#######>######>  ######> ##>   ##>###>   ##>#######>',
        '####> ####|##<====]##<==##>##<===##>##|   ##|####>  ##|##<====]',
        '##<####<##|#####>  ##|  ##|##|   ##|##|   ##|##<##> ##|#####>  ',
        '##|[##<]##|##<==]  ##|  ##|##|   ##|##|   ##|##|[##>##|##<==]  ',
        '##| [=] ##|#######>######<][######<][######<]##| [####|#######>',
        '[=]     [=][======][=====]  [=====]  [=====] [=]  [===][======]'
    )
    # Degrade horizontal violet -> bleu ; l'ombre est plus sombre pour l'effet relief
    $from = @(179, 79, 209); $to = @(54, 188, 247)
    $banner = @()
    foreach ($row in $letters) {
        $s = ""
        for ($x = 0; $x -lt $row.Length; $x++) {
            $ch = "$($row[$x])"
            if ($ch -eq ' ') { $s += ' '; continue }
            $t = $x / ($row.Length - 1)
            $k = if ($ch -eq '#') { 1 } else { 0.55 }
            $rgb = (0..2 | ForEach-Object { [int](($from[$_] + ($to[$_] - $from[$_]) * $t) * $k) }) -join ';'
            $s += "$e[38;2;${rgb}m" + $glyphs[$ch]
        }
        $banner += $s
    }
    $dot = [char]0x00B7
    $banner += ''
    $banner += "$e[1;38;2;230;237;243mMouhamedoune FALL $e[0;38;2;139;148;158m $dot  $e[38;2;54;188;247mFull-Stack Web & Mobile"
    $banner += "$e[38;2;139;148;158mgithub.com/pape-medoune"
    # Demi-blocs : chaque caractere affiche deux pixels superposes (haut / bas)
    $top = [char]0x2580; $bottom = [char]0x2584
    $width = ($pixels | Measure-Object -Property Length -Maximum).Maximum
    if ($pixels.Count % 2) { $pixels += '' }
    $rows = $pixels.Count / 2
    $offset = [math]::Floor(($rows - $banner.Count) / 2)
    for ($r = 0; $r -lt $rows; $r++) {
        $up = $pixels[2 * $r].PadRight($width)
        $down = $pixels[2 * $r + 1].PadRight($width)
        $line = "  "
        for ($x = 0; $x -lt $width; $x++) {
            $cu = $colors["$($up[$x])"]; $cd = $colors["$($down[$x])"]
            if ($cu -and $cd) { $line += "$e[38;2;${cu}m$e[48;2;${cd}m$top$e[0m" }
            elseif ($cu)      { $line += "$e[38;2;${cu}m$top$e[0m" }
            elseif ($cd)      { $line += "$e[38;2;${cd}m$bottom$e[0m" }
            else              { $line += " " }
        }
        $b = $r - $offset
        if ($b -ge 0 -and $b -lt $banner.Count) { $line += "    " + $banner[$b] }
        Write-Host "$line$e[0m"
    }
    Write-Host ""
}
Show-Avatar

# 2. Prompt moderne sur deux lignes, aux couleurs de l'avatar
#   +- [ pape-medoune ][ ~\Desktop\projet ][ branche master ]
#   +-> commande
function prompt {
    $ok = $?
    $e = [char]27
    $sep = [char]0xE0B0                      # separateur powerline (inclus dans Cascadia Mono)
    $branchIcon = [char]0xE0A0
    $purple = '179;79;209'; $blue = '54;188;247'; $slate = '48;54;61'; $light = '230;237;243'

    # Chemin raccourci : ~ pour le dossier utilisateur, 3 derniers dossiers au maximum
    $path = $ExecutionContext.SessionState.Path.CurrentLocation.Path
    if ($path.StartsWith($HOME, [StringComparison]::OrdinalIgnoreCase)) { $path = '~' + $path.Substring($HOME.Length) }
    $parts = $path -split '\\' | Where-Object { $_ }
    if ($parts.Count -gt 4) { $path = $parts[0] + '\' + [char]0x2026 + '\' + (($parts[-3..-1]) -join '\') }

    # Branche git, uniquement dans un depot
    $branch = $null
    if (Get-Command git -ErrorAction SilentlyContinue) {
        $branch = git rev-parse --abbrev-ref HEAD 2>$null
    }

    $corner1 = [char]0x256D; $corner2 = [char]0x2570; $dash = [char]0x2500; $arrow = [char]0x276F
    $s  = "`n$e[38;2;${purple}m$corner1$dash"
    $s += "$e[48;2;${purple}m$e[38;2;${light}m pape-medoune "
    $s += "$e[48;2;${slate}m$e[38;2;${purple}m$sep$e[38;2;${light}m $path "
    if ($branch) {
        $s += "$e[48;2;${blue}m$e[38;2;${slate}m$sep$e[38;2;13;17;23m $branchIcon $branch "
        $s += "$e[0m$e[38;2;${blue}m$sep"
    } else {
        $s += "$e[0m$e[38;2;${slate}m$sep"
    }
    $arrowColor = if ($ok) { $blue } else { '255;95;87' }
    $s += "`n$e[38;2;${purple}m$corner2$dash$e[38;2;${arrowColor}m$arrow$e[0m"
    Write-Host $s -NoNewline
    return " "
}
```

---

## 5. Tester

Ferme puis rouvre un onglet PowerShell dans Windows Terminal (ou tape `. $PROFILE`).
Tu dois voir l'avatar à gauche, la bannière MEDOUNE à droite, puis le prompt :

```
╭─ pape-medoune ▶ ~\…\Desktop\electroqc-iv1\pape-medoune ▶  master ▶
╰─❯
```

---

## 6. Comment ça marche (pour modifier plus tard)

### Avatar (`Show-Avatar`, tableau `$pixels`)
- Dessiné à partir de ma photo GitHub (pixel art 24×24), un caractère = un pixel :
  `#` contour · `H` cheveux · `.` peau · `o` yeux · `$` chaîne en or · espace = transparent.
- Couleurs RVB dans `$colors` (ex. cheveux `179;79;209`).
- **Demi-blocs** : chaque caractère affiché (`▀` / `▄`) montre 2 pixels superposés, ce qui divise la taille
  par deux sans déformer l'image.

### Bannière (`$letters`)
- Lettrage « MEDOUNE » en police figlet **ANSI Shadow**, encodé en ASCII :
  `#` → `█` · `=` → `═` · `|` → `║` · `<` → `╔` · `>` → `╗` · `[` → `╚` · `]` → `╝`.
- Dégradé horizontal de `$from = 179,79,209` (violet) à `$to = 54,188,247` (bleu) ;
  l'ombre est assombrie à 55 % pour l'effet relief.
- Pour changer le texte : `pip install pyfiglet` puis
  `python -c "import pyfiglet; print(pyfiglet.figlet_format('TEXTE', font='ansi_shadow'))"`,
  et convertir les symboles avec la légende ci-dessus.
- La signature (nom, titre, GitHub) est ajoutée dans les lignes `$banner += ...`.

### Prompt (`function prompt`)
- Ligne 1 : bloc violet `pape-medoune`, bloc gris avec le dossier courant, bloc bleu avec la branche git.
- Chemin : le dossier utilisateur devient `~` ; au-delà de 3 niveaux, le début est abrégé en `…`.
- Ligne 2 : flèche `❯` **bleue** si la dernière commande a réussi, **rouge** si elle a échoué.
- Couleurs : `$purple`, `$blue`, `$slate`, `$light` en haut de la fonction.

### Palette commune
| Rôle | RVB | Hex |
| :--- | :--- | :--- |
| Violet (cheveux, début du dégradé, bloc utilisateur) | 179;79;209 | `#B34FD1` |
| Bleu (fin du dégradé, branche, flèche) | 54;188;247 | `#36BCF7` |
| Gris ardoise (bloc dossier) | 48;54;61 | `#30363D` |
| Texte clair / contour avatar | 230;237;243 | `#E6EDF3` |
| Peau | 198;138;90 | `#C68A5A` |
| Or (chaîne) | 255;212;59 | `#FFD43B` |
| Rouge (erreur) | 255;95;87 | `#FF5F57` |

---

## 7. Dépannage

| Problème | Solution |
| :--- | :--- |
| Rien ne s'affiche au démarrage | Politique d'exécution : refaire l'étape 2 |
| Caractères bizarres (`Ã`, `â–`) | Le fichier contient de vrais symboles au lieu des `[char]0x....` : recoller le bloc tel quel |
| Couleurs approximatives | Utiliser Windows Terminal plutôt que l'ancienne console bleue |
| Carrés vides dans le prompt | La police du terminal a été changée : remettre **Cascadia Mono** (Paramètres → Profil → Apparence) |
| La bannière passe à la ligne | Élargir la fenêtre (environ 90 colonnes nécessaires) |
| Pas de branche affichée | Git non installé, ou dossier hors d'un dépôt git |

---

## 8. Revenir en arrière

Une sauvegarde de mon profil d'origine (avant l'avatar et la bannière) existe sur ce PC :
`C:\Users\User\OneDrive\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1.bak`
