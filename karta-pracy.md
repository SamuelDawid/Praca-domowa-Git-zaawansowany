# Zadania domowe -- Git zaawansowany

> **Cel pracy domowej:** Samodzielnie przećwiczyć wszystkie techniki z lekcji `27_git_zaawansowany.md` (konflikty, stash, cherry-pick, interactive rebase, reflog, bisect, reset/revert, GitHub flow). Każde zadanie wykonujesz **dwa razy**: raz w CLI (Git Bash), raz w IntelliJ IDEA -- żeby zobaczyć jak ta sama operacja wygląda w obu narzędziach.

---

## Jak korzystać z tych zadań

Każde zadanie zawiera:

* **Cel + ćwiczone narzędzia** -- co dokładnie ma się "wczepić w mięśnie".
* **Przygotowanie** -- gotowe komendy do utworzenia lokalnego katalogu i repo na GitHubie (krok po kroku).
* **Listing kodu / skrypt setupowy** -- jeden lub kilka bloków, które kopiujesz w całości i wykonujesz.
* **Scenariusz Wariant A (CLI / Git Bash)** -- numerowane kroki z gotowymi komendami.
* **Scenariusz Wariant B (IntelliJ IDEA)** -- ten sam efekt klikany w IDE.
* **Co ma być na GitHubie po zadaniu** -- checklist do sprawdzenia, czy zadanie jest skończone.
* **Pytania** -- odpowiedzi wpisujesz do pliku `RAPORT.md` w głównym repo `git-advanced-pd` (patrz "Sposób oddania pracy" na dole).

> **Ważne:** Każde zadanie ma **swoje osobne repo** -- nie mieszaj ich w jednym katalogu. Dzięki temu zawsze startujesz z czystego stanu i nie psujesz sobie poprzednich ćwiczeń.

> **Nie pomijaj wariantu IntelliJ.** Nawet jeśli wolisz CLI, zrób oba -- mapowanie "komenda CLI ↔ akcja w UI" jest najszybszą drogą do biegłości w Gicie.

---

## Spis treści

1. [Przygotowanie środowiska (jednorazowe)](#0-przygotowanie-środowiska-jednorazowe)
2. [Ściąga komend i skrótów](#ściąga-komend-i-skrótów)
3. [Zadanie 1 -- Pierwszy konflikt merge (CLI + IntelliJ + GitHub)](#zadanie-1-pierwszy-konflikt-merge-cli--intellij--github)
4. [Zadanie 2 -- Konflikt na rebase zamiast merge](#zadanie-2-konflikt-na-rebase-zamiast-merge)
5. [Zadanie 3 -- Git Stash w sytuacji "pilny hotfix"](#zadanie-3-git-stash-w-sytuacji-pilny-hotfix)
6. [Zadanie 4 -- Cherry-pick: przeniesienie commita między gałęziami](#zadanie-4-cherry-pick-przeniesienie-commita-między-gałęziami)
7. [Zadanie 5 -- Interactive Rebase: porządkowanie historii przed PR](#zadanie-5-interactive-rebase-porządkowanie-historii-przed-pr)
8. [Zadanie 6 -- Reflog: odzyskanie po `reset --hard`](#zadanie-6-reflog-odzyskanie-po-reset-hard)
9. [Zadanie 7 -- Bisect: binarne polowanie na buga](#zadanie-7-bisect-binarne-polowanie-na-buga)
10. [Zadanie 8 -- Reset (3 tryby) vs Revert](#zadanie-8-reset-3-tryby-vs-revert)
11. [Zadanie 9 -- Feature branch + PR z konfliktem (GitHub flow)](#zadanie-9-feature-branch--pr-z-konfliktem-github-flow)
12. [Zadanie 10 -- Śledztwo: kombinacja technik (samodzielne)](#zadanie-10-śledztwo-kombinacja-technik-samodzielne)
13. [Sposób oddania pracy](#sposób-oddania-pracy)

---

## 0. Przygotowanie środowiska (jednorazowe)

To rozdział, który robisz **raz** -- przed zadaniem 1. W kolejnych zadaniach zakłada się, że wszystko działa.

### 0.1. Sprawdź wersję Gita

W Git Bash (lub terminalu macOS / Linux):

```bash
git --version
```

Powinno pokazać `git version 2.23` lub nowsze. Jeśli starsze -- zaktualizuj Git ([https://git-scm.com/downloads](https://git-scm.com/downloads)).

### 0.2. Ustaw user.name i user.email

```bash
git config --global user.name "Imię Nazwisko"
git config --global user.email "twoj@email.com"
```

Sprawdź:

```bash
git config --global user.name
git config --global user.email
```

> **Uwaga:** ten sam e-mail powinien być powiązany z Twoim kontem GitHub (https://github.com/settings/emails) -- inaczej commity nie pokażą się jako Twoje.

### 0.3. Ustaw domyślną nazwę gałęzi głównej

```bash
git config --global init.defaultBranch main
```

### 0.4. Ustaw edytor (do interactive rebase i merge messages)

Wybierz jeden:

```bash
# Nano (najprostszy, terminalowy)
git config --global core.editor "nano"

# Visual Studio Code (jeśli masz zainstalowany)
git config --global core.editor "code --wait"
```

### 0.5. Konto na GitHubie + autoryzacja

* Załóż konto na [https://github.com/signup](https://github.com/signup) (jeśli jeszcze nie masz).
* Upewnij się, że `git push` działa. Są dwie opcje:

**Opcja A -- HTTPS + Personal Access Token (PAT)**

1. Wygeneruj token na [https://github.com/settings/tokens](https://github.com/settings/tokens) -- *Generate new token (classic)*, scope: `repo`, ważność: 90 dni.
2. Skopiuj token (pojawi się **tylko raz**).
3. Przy pierwszym `git push` Git zapyta o login i hasło -- jako hasło wklej token.
4. (Opcjonalnie) Zapamiętanie tokena: `git config --global credential.helper cache` (Linux) lub `git config --global credential.helper osxkeychain` (macOS) lub `git config --global credential.helper manager` (Windows).

**Opcja B -- SSH key**

1. Wygeneruj klucz: `ssh-keygen -t ed25519 -C "twoj@email.com"`. Naciśnij Enter na wszystkich pytaniach.
2. Skopiuj klucz publiczny: `cat ~/.ssh/id_ed25519.pub` (zaznacz i skopiuj wynik).
3. Dodaj na [https://github.com/settings/ssh/new](https://github.com/settings/ssh/new) -- wklej klucz, daj nazwę, *Add SSH key*.
4. W każdym repo używaj URL `git@github.com:<TWOJ-USER>/<repo>.git` zamiast `https://github.com/...`.

> **Pełna instrukcja GitHuba:** [https://docs.github.com/en/authentication](https://docs.github.com/en/authentication)

### 0.6. Sprawdź, że IntelliJ widzi Git

W IntelliJ:

1. *Settings → Version Control → Git*.
2. Pole **Path to Git executable** -- powinno mieć ścieżkę (np. `/usr/bin/git` lub `C:/Program Files/Git/cmd/git.exe`).
3. Kliknij **Test** -- powinno pokazać wersję Gita.

### 0.7. Utwórz główne repo `git-advanced-pd` na raport

To jedno repo, do którego wrzucisz finalny `RAPORT.md` po wszystkich zadaniach (patrz [Sposób oddania pracy](#sposób-oddania-pracy)). Repo na zadania 1-10 to osobne repo (każde zadanie = osobne repo).

```bash
mkdir git-advanced-pd && cd git-advanced-pd
git init
echo "# Praca domowa: Git zaawansowany" > README.md
git add README.md
git commit -m "init: praca domowa Git zaawansowany"
```

Następnie utwórz repo na GitHubie i podłącz:

1. [https://github.com/new](https://github.com/new) → nazwa: `git-advanced-pd`, Public, **NIE** zaznaczaj "Add README".
2. *Create repository*.
3. Skopiuj URL (HTTPS lub SSH) i:

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-advanced-pd.git
git branch -M main
git push -u origin main
```

> **W całym dokumencie zastępuj `<TWOJ-USER>` swoim loginem GitHub.**

---

## Ściąga komend i skrótów

### Komendy CLI (najczęściej używane)

```bash
# === STAN ===
git status                          # co jest zmienione, w stagingu, untracked
git log --oneline                   # zwięzła historia
git log --oneline --graph --all     # historia z gałęziami
git reflog                          # dziennik wszystkich operacji HEAD

# === GAŁĘZIE ===
git switch <branch>                 # przełącz gałąź
git switch -c <branch>              # utwórz i przełącz
git branch -d <branch>              # usuń (bezpiecznie)
git branch -D <branch>              # usuń (siłowo)

# === POŁĄCZENIA ===
git merge <branch>                  # merge (może wywołać konflikt)
git rebase <branch>                 # rebase (może wywołać konflikt)
git rebase -i HEAD~N                # interactive rebase ostatnich N commitów
git cherry-pick <hash>              # skopiuj jeden commit

# === SCHOWEK ===
git stash push -m "opis"            # schowaj zmiany
git stash list                      # lista
git stash pop                       # wyciągnij ostatni (usuwa z listy)
git stash apply stash@{N}           # wyciągnij N-ty (zostaw)
git stash drop                      # usuń ostatni
git stash -u                        # uwzględnij untracked

# === COFANIE ===
git reset --soft HEAD~1             # cofnij commit, zmiany w staging
git reset HEAD~1                    # cofnij commit, zmiany w working dir
git reset --hard HEAD~1             # cofnij commit + wymaż zmiany (DESTRUCTIVE!)
git revert <hash>                   # nowy commit cofający

# === SZUKANIE BUGA ===
git bisect start
git bisect bad                      # obecny zły
git bisect good <hash>              # ten był dobry
git bisect reset                    # zakończ

# === ZDALNE ===
git push                            # zwykły push
git push --force-with-lease         # bezpieczny force push (po rebase)
git fetch <remote>                  # pobierz bez merge
git pull                            # fetch + merge
```

### Skróty IntelliJ IDEA (Git)

| Akcja | Windows/Linux | macOS |
|-------|---------------|-------|
| Otwórz Git Tool Window | `Alt+9` | `⌘9` |
| Otwórz Commit | `Ctrl+K` | `⌘K` |
| Push | `Ctrl+Shift+K` | `⌘⇧K` |
| Update Project (pull/fetch) | `Ctrl+T` | `⌘T` |
| Otwórz Terminal IntelliJ | `Alt+F12` | `⌥F12` |
| Annotate / Git Blame | prawy klik na marginesie linii → *Annotate with Git Blame* | jw. |
| Git Log | *Git → Show Git Log* | jw. |

### Mapowanie CLI ↔ IntelliJ (najważniejsze)

| Komenda CLI | Akcja IntelliJ |
|-------------|----------------|
| `git switch <b>` | *Branches* (prawy dolny róg) → wybór gałęzi → *Checkout* |
| `git switch -c <b>` | *Branches* → *New Branch* |
| `git merge <b>` | *Git → Merge…* |
| `git rebase <b>` | *Git → Rebase Current Branch onto…* |
| `git rebase -i HEAD~N` | *Git → Log* → prawy klik na N-tym commicie → *Interactively Rebase from Here…* |
| `git cherry-pick <hash>` | *Git → Log* → prawy klik na commit → *Cherry-Pick* |
| `git stash` | *Git → Uncommitted Changes → Stash Changes…* |
| `git stash pop` | *Git → Uncommitted Changes → Unstash Changes…* |
| `git reset --hard <hash>` | *Git → Log* → prawy klik na commit → *Reset Current Branch to Here…* → tryb Hard |
| `git revert <hash>` | *Git → Log* → prawy klik na commit → *Revert Commit* |
| `git push --force-with-lease` | *Git → Push…* → strzałka obok przycisku Push → *Force Push* (IntelliJ domyślnie używa `--force-with-lease`) |
| `git reflog` | **Brak natywnego widoku!** Otwórz Terminal IntelliJ (`Alt+F12`) i wpisz `git reflog`. |
| `git bisect ...` | **Brak natywnego widoku!** Użyj Terminala IntelliJ (`Alt+F12`). |

---

## Zadanie 1 -- Pierwszy konflikt merge (CLI + IntelliJ + GitHub)

**Ćwiczone narzędzia:** rozwiązywanie konfliktu (markery `<<<<<<<` / `=======` / `>>>>>>>`), narzędzie 3-panelowe IntelliJ, GitHub Web Editor "Resolve conflicts".

**Opis:** Utworzysz repo z plikiem `Menu.java` i dwoma gałęziami, które dodają **różne pozycje menu w tej samej linii**. Spróbujesz zmergować je trzema sposobami: CLI, IntelliJ, GitHub UI. Porównasz efekty.

### Przygotowanie

#### Lokalnie

```bash
mkdir git-zad1-konflikty && cd git-zad1-konflikty
git init
git branch -M main
```

#### Na GitHubie

1. Otwórz [https://github.com/new](https://github.com/new).
2. Repository name: `git-zad1-konflikty`.
3. Visibility: **Public**.
4. **NIE** zaznaczaj "Add a README file".
5. Kliknij **Create repository**.
6. W lokalnym katalogu wykonaj (zastąp `<TWOJ-USER>`):

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-zad1-konflikty.git
```

### Skrypt setupowy (skopiuj w całości i wklej w Git Bash)

```bash
# Plik bazowy Menu.java
cat > Menu.java << 'EOF'
public class Menu {
    public static void main(String[] args) {
        System.out.println("=== MENU RESTAURACJI ===");
        System.out.println("1. Pizza Margherita");
        System.out.println("2. Burger klasyczny");
    }
}
EOF

git add Menu.java
git commit -m "feat: add basic restaurant menu"
git push -u origin main

# Gałąź feature/add-pasta -- dodaje Pastę w linii 6
git switch -c feature/add-pasta
cat > Menu.java << 'EOF'
public class Menu {
    public static void main(String[] args) {
        System.out.println("=== MENU RESTAURACJI ===");
        System.out.println("1. Pizza Margherita");
        System.out.println("2. Burger klasyczny");
        System.out.println("3. Pasta Carbonara");
    }
}
EOF
git add Menu.java
git commit -m "feat: add pasta to menu"
git push -u origin feature/add-pasta

# Wracamy na main
git switch main

# Gałąź feature/add-salad -- dodaje Sałatkę w TEJ SAMEJ linii 6
git switch -c feature/add-salad
cat > Menu.java << 'EOF'
public class Menu {
    public static void main(String[] args) {
        System.out.println("=== MENU RESTAURACJI ===");
        System.out.println("1. Pizza Margherita");
        System.out.println("2. Burger klasyczny");
        System.out.println("3. Salatka Cezar");
    }
}
EOF
git add Menu.java
git commit -m "feat: add salad to menu"
git push -u origin feature/add-salad

git switch main
git log --oneline --all --graph
```

Po skrypcie masz: `main` z bazowym menu, `feature/add-pasta` z pastą, `feature/add-salad` z sałatką -- obie gałęzie zmieniają **tę samą linię**.

### Scenariusz -- Wariant A (CLI / Git Bash)

1. Najpierw mergujemy `feature/add-pasta` -- bez konfliktu:

   ```bash
   git switch main
   git merge feature/add-pasta
   git push
   ```

2. Próbujemy zmergować `feature/add-salad`:

   ```bash
   git merge feature/add-salad
   ```

   Git wypisze:
   ```
   Auto-merging Menu.java
   CONFLICT (content): Merge conflict in Menu.java
   Automatic merge failed; fix conflicts and then commit the result.
   ```

3. Otwórz `Menu.java` w edytorze (np. `nano Menu.java` lub `code Menu.java`). Zobaczysz:

   ```
   <<<<<<< HEAD
           System.out.println("3. Pasta Carbonara");
   =======
           System.out.println("3. Salatka Cezar");
   >>>>>>> feature/add-salad
   ```

4. **Edytuj plik tak, żeby zachować obie pozycje** (usuń markery):

   ```java
           System.out.println("3. Pasta Carbonara");
           System.out.println("4. Salatka Cezar");
   ```

5. Zapisz, a następnie:

   ```bash
   git add Menu.java
   git commit                    # Git wygeneruje wiadomość "Merge branch ..."
   git push
   ```

6. Sprawdź:

   ```bash
   git log --oneline --graph --all
   ```

### Scenariusz -- Wariant B (IntelliJ IDEA)

> **Reset przed wariantem B** -- żeby zacząć od zera bez zmiany skryptu, w CLI wykonaj:
>
> ```bash
> # 1) Znajdź hash bazowego commitu (sprzed jakichkolwiek merge):
> git log --oneline
> # Skopiuj hash commitu "feat: add basic restaurant menu"
>
> # 2) Zresetuj main do stanu bazowego:
> git reset --hard <hash-bazowego-menu>   # albo równoważnie: git reset --hard origin/main~2
> git push --force-with-lease             # zsynchronizuj remote
> ```
>
> **Uwaga:** `git reset --hard origin/main` **nie zadziała** w tym miejscu -- po pushu z Wariantu A `origin/main` zawiera już merge commit, więc reset cofnąłby lokalnego `main` do dokładnie tego samego stanu (z mergiem). Trzeba cofnąć się do commitu **przed** mergami -- stąd `origin/main~2` (dwa kroki wstecz po pierwszym rodzicu: merge sałatki → tip pasty po fast-forward → commit bazowy) lub bezpośrednio hash bazowego menu.

1. Otwórz katalog `git-zad1-konflikty` w IntelliJ (*File → Open*).
2. Prawy dolny róg → **Branches** → kliknij `feature/add-pasta` → **Checkout**.
3. Otwórz **Branches** → kliknij `main` → **Checkout**.
4. *Git → Merge…* → wybierz `feature/add-pasta` → **Merge**. Powinno przejść bez konfliktu. *Git → Push* (`Ctrl+Shift+K`).
5. *Git → Merge…* → wybierz `feature/add-salad` → **Merge**.
6. IntelliJ pokaże okno **Merge Conflicts** z listą plików. Kliknij **Merge** przy `Menu.java`.
7. Otworzy się **narzędzie 3-panelowe**:
    * **LEFT** -- Twoja wersja (Pasta).
    * **RIGHT** -- Ich wersja (Salatka).
    * **CENTER** -- wynik (puste markery do uzupełnienia).
8. Kliknij **>>** przy LEFT (wstawia Pastę do CENTER) i **<<** przy RIGHT (wstawia Sałatkę). Edytuj numery (3, 4).
9. Kliknij **Apply**.
10. IntelliJ otworzy okno commit -- napisz wiadomość, *Commit*. Push (`Ctrl+Shift+K`).

### Scenariusz -- Wariant C (GitHub Web Editor)

> **Reset przed wariantem C** w CLI:
>
> ```bash
> # 1) Znajdź hash bazowego commitu (sprzed jakichkolwiek merge):
> git log --oneline
> # Skopiuj hash commitu "feat: add basic restaurant menu"
>
> # 2) Zresetuj main do stanu bazowego:
> git reset --hard <hash-bazowego-menu>   # albo równoważnie: git reset --hard origin/main~2
> git push --force-with-lease
> ```
>
> **Uwaga:** podobnie jak przy Wariancie B -- `git reset --hard origin/main` tu nie zadziała, bo `origin/main` po Wariancie A/B zawiera już merge commit. Trzeba cofnąć się do commitu **przed** mergami.

1. Otwórz repo na GitHubie.
2. *Pull requests → New pull request* → base: `main`, compare: `feature/add-salad` → **Create pull request**.
3. Na stronie PR zobaczysz: *"This branch has conflicts that must be resolved"* + przycisk **Resolve conflicts**.
4. Kliknij *Resolve conflicts*. GitHub otworzy edytor z markerami `<<<<<<<` / `=======` / `>>>>>>>` -- ręcznie usuń markery, zostaw obie pozycje.
5. Kliknij **Mark as resolved** → **Commit merge**.
6. Wróć do PR → **Merge pull request** → **Confirm merge**.

### Co ma być na GitHubie po zadaniu

* [ ] Repo `git-zad1-konflikty` publiczne.
* [ ] Gałęzie: `main`, `feature/add-pasta`, `feature/add-salad`.
* [ ] Co najmniej **jeden** PR zmergowany w UI GitHuba (po wariancie C lub B+push).
* [ ] `git log --graph --oneline --all` na `main` pokazuje merge commit łączący `feature/add-salad`.

### Pytania (do RAPORT.md)

1. Jaki był hash merge commitu w wariancie A (CLI)? A w wariancie B (IntelliJ)? Czy są takie same?
2. Jaką domyślną wiadomość commit wygenerował Git w wariancie A (CLI)? A IntelliJ w wariancie B?
3. Kiedy najwygodniej rozwiązywać konflikt w UI GitHuba (wariant C), a kiedy lepiej zrobić to lokalnie?
4. Wymień 2 powody, dla których IntelliJ 3-panel jest wygodniejszy niż edycja markerów ręcznie.
5. Załącz screenshot 3-panelowego narzędzia IntelliJ podczas rozwiązywania konfliktu.

---

## Zadanie 2 -- Konflikt na rebase zamiast merge

**Ćwiczone narzędzia:** `git rebase`, `git rebase --continue` / `--abort`, push z `--force-with-lease` po rebase.

**Opis:** Powtórzysz konflikt z zadania 1, ale rozwiążesz go przez **rebase** zamiast merge -- żeby zobaczyć różnicę w historii (linia prosta vs merge commit).

### Przygotowanie

#### Lokalnie

```bash
mkdir git-zad2-rebase-konflikt && cd git-zad2-rebase-konflikt
git init
git branch -M main
```

#### Na GitHubie

1. [https://github.com/new](https://github.com/new) → nazwa: `git-zad2-rebase-konflikt`, Public, BEZ README.
2. *Create repository*.
3. Lokalnie:

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-zad2-rebase-konflikt.git
```

### Skrypt setupowy

```bash
# Plik bazowy README.md
cat > README.md << 'EOF'
# Projekt TODO

- [ ] Zadanie 1: napisz funkcję sumującą
- [ ] Zadanie 2: dodaj testy jednostkowe
- [ ] Zadanie 3: wdróż na produkcję
EOF

git add README.md
git commit -m "init: lista TODO"
git push -u origin main

# Gałąź feature/task-A -- modyfikuje linię 3 (Zadanie 1)
git switch -c feature/task-A
sed -i.bak 's|Zadanie 1: napisz funkcję sumującą|Zadanie 1: napisz funkcję mnożącą|' README.md && rm README.md.bak
git add README.md
git commit -m "feat: zmiana zadania 1 na mnożenie"
git push -u origin feature/task-A

# Wracamy na main i robimy zmianę w TEJ SAMEJ linii
git switch main
sed -i.bak 's|Zadanie 1: napisz funkcję sumującą|Zadanie 1: napisz funkcję dzielącą|' README.md && rm README.md.bak
git add README.md
git commit -m "feat: zmiana zadania 1 na dzielenie"
git push

git log --oneline --all --graph
```

Teraz `main` i `feature/task-A` mają **rozbieżną historię** -- każda zmienia tę samą linię.

### Scenariusz -- Wariant A (CLI / Git Bash)

1. Przełącz się na feature i spróbuj rebase:

   ```bash
   git switch feature/task-A
   git rebase main
   ```

   Git wypisze:
   ```
   Auto-merging README.md
   CONFLICT (content): Merge conflict in README.md
   error: could not apply <hash>... feat: zmiana zadania 1 na mnożenie
   ```

2. Sprawdź stan:

   ```bash
   git status
   ```

   Zobaczysz: *"You are currently rebasing branch ..."*.

3. Otwórz `README.md`, zobacz markery `<<<<<<< HEAD ... ======= ... >>>>>>>`. Wybierz wersję mnożenia (Twoja zmiana z feature) i usuń markery.

4. Kontynuuj rebase:

   ```bash
   git add README.md
   git rebase --continue
   ```

5. Push -- ponieważ historia zmieniła swój kształt, musisz użyć `--force-with-lease`:

   ```bash
   git push --force-with-lease
   ```

6. Zobacz historię:

   ```bash
   git log --oneline --all --graph
   ```

   Zauważysz: **brak merge commitu**. `feature/task-A` jest "po" main, w prostej linii.

7. **Bonus:** spróbuj `--abort`. Najpierw przywróć stan sprzed rebase:

   ```bash
   git switch feature/task-A
   # Po force-pushu w kroku 5 origin/feature/task-A wskazuje już na rebase'owany commit.
   # Git po rebase'ie zapisuje stan sprzed operacji w ORIG_HEAD -- używamy tego:
   git reset --hard ORIG_HEAD
   git push --force-with-lease               # zsynchronizuj remote z oryginalnym stanem
   ```

   ...następnie powtórz `git rebase main`, aż dojdziesz do konfliktu, i:

   ```bash
   git rebase --abort
   ```

   Stan wraca do tego sprzed rebase.

### Scenariusz -- Wariant B (IntelliJ IDEA)

> **Reset przed wariantem B:**
>
> ```bash
> git switch feature/task-A
>
> # Najprościej -- git po rebase'ie zapisuje stan sprzed operacji w ORIG_HEAD:
> git reset --hard ORIG_HEAD
> git push --force-with-lease                # zsynchronizuj remote z oryginalnym stanem
> ```
>
> Jeśli `ORIG_HEAD` zostało już nadpisane przez inną operację (np. drugi rebase albo merge), użyj reflogu:
>
> ```bash
> git reflog
> # Znajdź wpis sprzed rebase, np.:
> #   <hash> HEAD@{N}: commit: feat: zmiana zadania 1 na mnożenie
> git reset --hard HEAD@{N}                  # N = numer wpisu sprzed rebase
> git push --force-with-lease
> ```
>
> **Uwaga:** `git reset --hard origin/feature/task-A` ani `... origin/feature/task-A^` **nie zadziałają** w tym miejscu -- po force-pushu z Wariantu A `origin/feature/task-A` wskazuje na rebase'owany commit, a jego pierwszym rodzicem jest tip gałęzi `main` (commit "dzielenie"). `ORIG_HEAD` lub reflog to jedyne źródła prawdy o stanie sprzed rebase'a.

1. Otwórz katalog w IntelliJ.
2. Branches → checkout `feature/task-A`.
3. *Git → Rebase Current Branch onto…* → wybierz `main` → **Rebase**.
4. IntelliJ pokaże okno **Conflicts** -- kliknij **Merge** przy `README.md`. Otworzy się 3-panel.
5. W 3-panelu wybierz wersję zawierającą **mnożenie** -- najpewniejszą wskazówką jest **nazwa gałęzi w nagłówku panelu** (`feature/task-A` = mnożenie, `main` = dzielenie). Uwaga: w niektórych wersjach IntelliJ podczas rebase'a układ LEFT/RIGHT bywa odwrócony względem klasycznego merge'a, więc nie kieruj się stroną panelu, tylko nazwą branchu. Po wyborze → **Apply**.
6. IntelliJ kontynuuje rebase automatycznie.
7. *Git → Push…* → strzałka obok **Push** → **Force Push** (IntelliJ użyje `--force-with-lease`).

### Co ma być na GitHubie po zadaniu

* [ ] Repo `git-zad2-rebase-konflikt` publiczne.
* [ ] Gałąź `main` z commitem "dzielenie".
* [ ] Gałąź `feature/task-A` z commitem "mnożenie" **rebasewanym** na main (po tej drugiej, w prostej linii).
* [ ] `git log --graph --oneline --all` pokazuje **prostą historię bez merge commitu**.

### Pytania (do RAPORT.md)

1. Jak wygląda `git log --graph --oneline --all` po rebase? Załącz screenshot.
2. Jaka jest różnica w historii między rozwiązaniem przez merge (zad. 1) a rebase (zad. 2)?
3. Dlaczego po rebase trzeba użyć `git push --force-with-lease`? Co by się stało gdybyś użył zwykłego `git push`?
4. Czym różni się `--force-with-lease` od `--force`? Kiedy używać którego?
5. Co robi `git rebase --abort`? Kiedy chcesz tego użyć?

---

## Zadanie 3 -- Git Stash w sytuacji "pilny hotfix"

**Ćwiczone narzędzia:** `git stash push`, `git stash list`, `git stash pop`, `git stash apply stash@{N}`, `git stash drop`, `git stash -u`.

**Opis:** Pracujesz na feature branchu, masz niezacommitowane zmiany. Nagle musisz pilnie naprawić buga na produkcji. Schowasz zmiany przez `stash`, naprawisz buga, wrócisz, przywrócisz zmiany.

### Przygotowanie

#### Lokalnie

```bash
mkdir git-zad3-stash && cd git-zad3-stash
git init
git branch -M main
```

#### Na GitHubie

1. [https://github.com/new](https://github.com/new) → `git-zad3-stash`, Public, BEZ README.
2. *Create repository*.
3. Lokalnie:

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-zad3-stash.git
```

### Skrypt setupowy

```bash
cat > UserProfile.java << 'EOF'
package com.example;

public class UserProfile {
    private String name;
    private String email;

    public UserProfile(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public String getName() { return name; }
    public String getEmail() { return email; }
}
EOF

git add UserProfile.java
git commit -m "feat: add UserProfile class"
git push -u origin main

git switch -c feature/user-profile
```

### Scenariusz -- Wariant A (CLI / Git Bash)

1. Wprowadź **niedokończoną** zmianę w `UserProfile.java` (dodaj nowe pole, ale nie kompletnie):

   ```bash
   cat > UserProfile.java << 'EOF'
   package com.example;

   public class UserProfile {
       private String name;
       private String email;
       private int age;     // niedokończone -- brak gettera, settera, kosntruktora!

       public UserProfile(String name, String email) {
           this.name = name;
           this.email = email;
       }

       public String getName() { return name; }
       public String getEmail() { return email; }
   }
   EOF
   ```

2. Sprawdź `git status` -- masz zmodyfikowany plik bez commita.

3. Schowaj zmiany:

   ```bash
   git stash push -m "WIP: dodaję pole age"
   git status     # powinno być "nothing to commit, working tree clean"
   ```

4. **Symulacja hotfix** -- przejdź na main, dodaj nowy plik, commituj, push:

   ```bash
   git switch main
   git switch -c hotfix/security-fix
   cat > SecurityFix.java << 'EOF'
   package com.example;

   public class SecurityFix {
       public static void patch() {
           System.out.println("Patched CVE-2025-1234");
       }
   }
   EOF
   git add SecurityFix.java
   git commit -m "fix: critical security patch"
   git push -u origin hotfix/security-fix

   # Mergujemy hotfix do main (symulacja zmergowania PR)
   git switch main
   git merge hotfix/security-fix
   git push
   ```

5. Wróć na feature i przywróć stash:

   ```bash
   git switch feature/user-profile
   git stash list                   # zobacz że stash istnieje
   git stash pop                    # wyciągnij i usuń z listy
   git status                       # zmiany wróciły!
   ```

6. Dokończ pracę -- dodaj getter/setter/konstruktor dla `age`:

   ```bash
   cat > UserProfile.java << 'EOF'
   package com.example;

   public class UserProfile {
       private String name;
       private String email;
       private int age;

       public UserProfile(String name, String email, int age) {
           this.name = name;
           this.email = email;
           this.age = age;
       }

       public String getName() { return name; }
       public String getEmail() { return email; }
       public int getAge() { return age; }
   }
   EOF
   git add UserProfile.java
   git commit -m "feat: add age field with getter and constructor param"
   git push -u origin feature/user-profile
   ```

7. **Mini-eksperyment z dwoma stashami:**

   ```bash
   echo "// TODO 1" >> UserProfile.java
   git stash push -m "stash A"

   echo "// TODO 2" >> UserProfile.java
   git stash push -m "stash B"

   git stash list
   # stash@{0}: On feature/user-profile: stash B
   # stash@{1}: On feature/user-profile: stash A

   git stash apply stash@{1}        # wyciągnij stash A, ale go nie usuwaj
   git stash list                   # nadal masz oba!
   git stash drop stash@{0}         # usuń stash B
   git stash drop                   # usuń pozostały stash (A)
   git checkout -- UserProfile.java # cofnij zmiany w pliku
   ```

8. **Mini-bonus -- untracked files:**

   ```bash
   echo "secret" > nowy_plik.txt    # nowy plik, nie dodany do gita
   git stash                        # czy nowy_plik.txt zniknął?
   ls nowy_plik.txt                 # JEST! stash domyślnie nie chowa untracked.
   git stash -u                     # tym razem zniknie
   ls nowy_plik.txt                 # brak
   git stash pop                    # wraca
   rm nowy_plik.txt                 # sprzątanie
   ```

### Scenariusz -- Wariant B (IntelliJ IDEA)

1. Otwórz katalog w IntelliJ. Branches → checkout `feature/user-profile`.
2. Wprowadź **niedokończoną zmianę** w `UserProfile.java` (jak w kroku 1 wariantu A) -- ręcznie w edytorze IntelliJ.
3. *Git → Uncommitted Changes → Stash Changes…* → Message: `WIP: dodaję pole age` → **Create Stash**.
4. Branches → checkout `main` → *Branches → New Branch from Current* → `hotfix/security-fix`.
5. *File → New → Java Class* → nazwa: `SecurityFix`. Wklej kod jak wyżej.
6. *Git → Commit* (`Ctrl+K`) → wiadomość: `fix: critical security patch` → **Commit and Push**.
7. Branches → checkout `main`. *Git → Merge…* → `hotfix/security-fix` → **Merge** → push.
8. Branches → checkout `feature/user-profile`.
9. *Git → Uncommitted Changes → Unstash Changes…* → wybierz stash → **Apply Stash** (lub **Pop Stash**).
10. Dokończ pracę nad `UserProfile.java`, commit, push.

### Co ma być na GitHubie po zadaniu

* [ ] Repo `git-zad3-stash` publiczne.
* [ ] Gałąź `main` zawiera `SecurityFix.java`.
* [ ] Gałąź `feature/user-profile` ma `UserProfile.java` z polem `age` + getter + parametr konstruktora.
* [ ] Gałąź `hotfix/security-fix` istnieje (bez force-delete).

### Pytania (do RAPORT.md)

1. Jaka jest różnica między `git stash pop` a `git stash apply`? Kiedy chcesz użyć którego?
2. Co się dzieje gdy masz 2 stashe i robisz `git stash pop`? Który zostanie wyciągnięty?
3. Dlaczego `git stash` domyślnie **nie chowa** untracked files? Kiedy to jest problem?
4. Czy stash idzie na remote po `git push`? Sprawdź na GitHubie -- czy widzisz tam stashe?
5. Załącz screenshot `git stash list` z momentu, gdy miałeś 2 stashe.

---

## Zadanie 4 -- Cherry-pick: przeniesienie commita między gałęziami

**Ćwiczone narzędzia:** `git cherry-pick <hash>`, `git cherry-pick <h1>..<h2>` (zakres), `git cherry-pick --continue` / `--abort`.

**Opis:** Masz feature branch z 3 commitami. Chcesz przenieść **tylko drugi** commit na main, bez merge'owania całej gałęzi. Zauważysz, że cherry-pickowany commit ma **inny hash** niż oryginał.

### Przygotowanie

#### Lokalnie

```bash
mkdir git-zad4-cherry-pick && cd git-zad4-cherry-pick
git init
git branch -M main
```

#### Na GitHubie

1. [https://github.com/new](https://github.com/new) → `git-zad4-cherry-pick`, Public, BEZ README.
2. Lokalnie:

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-zad4-cherry-pick.git
```

### Skrypt setupowy

```bash
cat > app.txt << 'EOF'
Linia 1: początek

=== Sekcja A ===
[TODO A]

=== Sekcja B ===
[TODO B]

=== Sekcja C ===
[TODO C]
EOF
git add app.txt
git commit -m "init: szkielet z 3 sekcjami"
git push -u origin main

git switch -c feature/A

# Commit 1 -- wypełnia sekcję A
sed -i.bak 's|\[TODO A\]|Sekcja A: zmiana w feature/A (commit 1)|' app.txt && rm app.txt.bak
git add app.txt
git commit -m "feat: sekcja A (commit 1 na feature/A)"

# Commit 2 -- wypełnia sekcję B -- TEN przeniesiemy
sed -i.bak 's|\[TODO B\]|Sekcja B: ważna zmiana w feature/A (commit 2)|' app.txt && rm app.txt.bak
git add app.txt
git commit -m "feat: sekcja B (commit 2 na feature/A) -- TEN przeniesiemy"

# Commit 3 -- wypełnia sekcję C
sed -i.bak 's|\[TODO C\]|Sekcja C: dodatek w feature/A (commit 3)|' app.txt && rm app.txt.bak
git add app.txt
git commit -m "feat: sekcja C (commit 3 na feature/A)"

git push -u origin feature/A

git log --oneline feature/A
```

> **Dlaczego sekcje z pustymi liniami?** Cherry-pick to nie zwykły "wklej commit" -- to **3-way merge** diff'u tego commitu z aktualną gałęzią. Jeśli zmiany w gałęzi `feature/A` byłyby na sąsiednich liniach (np. każdy commit appenduje do tej samej listy), to cherry-pickując środkowy commit, git zobaczyłby konflikt z brakującym kontekstem (linią dodaną przez commit 1). Rozdzielenie sekcji pustymi liniami sprawia, że każda zmiana jest osobnym "hunk'iem" i cherry-pick przechodzi bezkonfliktowo.

Zanotuj **hash drugiego commita** ("sekcja B") -- będzie potrzebny.

### Scenariusz -- Wariant A (CLI / Git Bash)

1. Wyświetl historię i znajdź hash drugiego commita:

   ```bash
   git log --oneline feature/A
   # np.
   # f4e5d6a feat: sekcja C (commit 3 na feature/A)
   # b3c4d5e feat: sekcja B (commit 2 na feature/A) -- TEN przeniesiemy
   # a1b2c3d feat: sekcja A (commit 1 na feature/A)
   # ...
   ```

2. Przełącz się na main i cherry-pick:

   ```bash
   git switch main
   git cherry-pick b3c4d5e         # zamień na swój hash
   ```

3. Sprawdź:

   ```bash
   cat app.txt
   git log --oneline
   ```

   `app.txt` na main ma wypełnioną **sekcję B** (z commitu 2), natomiast sekcje A i C nadal mają placeholdery `[TODO A]` / `[TODO C]` -- commity 1 i 3 z feature/A **nie zostały skopiowane**.

4. Porównaj hashe:

   ```bash
   git log --oneline                # commit na main -- nowy hash
   git log --oneline feature/A      # ten sam commit na feature/A -- stary hash
   ```

   **Hashe są różne!** To dwa fizycznie różne commity z tą samą zawartością.

5. Push:

   ```bash
   git push
   ```

### Reset między wariantami A i B -- przywrócenie stanu początkowego

Po wykonaniu wariantu CLI `main` zawiera już commit "sekcja B" (cherry-picked). Żeby zrobić wariant IntelliJ od nowa na tym samym repo, musisz cofnąć `main` do **pierwszego** commita (`init: szkielet z 3 sekcjami`) -- **NIE do wierzchołka `feature/A`**.

```bash
git switch main
git log --oneline --all              # znajdź hash commita "init: szkielet z 3 sekcjami"
git reset --hard <hash-init>         # np. git reset --hard a1b2c3d
cat app.txt                          # SPRAWDŹ: powinny być wszystkie [TODO A], [TODO B], [TODO C]
git push --force-with-lease origin main
```

> **Pułapka -- DLACZEGO to ważne:** jeśli zamiast hasha "init" użyjesz hasha wierzchołka `feature/A` (np. commitu "sekcja C"), to przesuniesz `main` na koniec `feature/A`. Wtedy `main` będzie już zawierał wszystkie zmiany z feature'a, a cherry-pick "sekcja B" zwróci komunikat:
>
> `Nothing to cherry-pick -- All changes from [commit] have already been applied`
>
> To nie jest błąd Gita -- to logiczna konsekwencja. Cherry-pick stosuje **diff** commita, a skoro te zmiany już są na gałęzi, nie ma czego stosować. **Zawsze sprawdź `cat app.txt`** -- musisz zobaczyć trzy `[TODO ...]` zanim ruszysz z wariantem B.

### Scenariusz -- Wariant B (IntelliJ IDEA)

1. Otwórz katalog w IntelliJ.
2. *Git → Show Git Log* (`Alt+9`) -- po lewej zaznacz gałąź `feature/A`.
3. Znajdź drugi commit ("sekcja B"). Prawy klik → **Cherry-Pick**.
4. IntelliJ pokaże dialog commit -- przejrzyj wiadomość, kliknij **Apply**.
5. Sprawdź w *Git → Log* -- teraz oba commity (oryginał na feature/A i cherry-picked na main) widać side-by-side.
6. *Git → Push* (`Ctrl+Shift+K`).

### Co ma być na GitHubie po zadaniu

* [ ] Repo `git-zad4-cherry-pick` publiczne.
* [ ] Gałąź `main` zawiera **commit cherry-pickowany** ("sekcja B"), ale **nie** zawiera commitów 1 i 3 z feature/A.
* [ ] Gałąź `feature/A` zawiera oryginalne 3 commity.
* [ ] Hashe commitu na main (cherry-pick) i feature/A (oryginał) są **różne**.

### Pytania (do RAPORT.md)

1. Jaki był hash oryginalnego commita na `feature/A`? Jaki na `main` po cherry-pick? Załącz oba `git log --oneline`.
2. Dlaczego hash jest różny, jeśli treść (diff) jest identyczna? (Wskazówka: parent commit, timestamp.)
3. Co zawiera `app.txt` na main po cherry-pick (która sekcja jest wypełniona, a które mają nadal `[TODO ...]`)? Czego tam **nie ma**?
4. Co się dzieje, gdy cherry-pick wywoła konflikt? Jak go rozwiązać (analogia do merge/rebase)? **Eksperyment:** spróbuj cherry-pickować trzeci commit (zmiana sekcji C) -- czy też przejdzie bezkonfliktowo? Dlaczego?
5. Kiedy używać cherry-pick zamiast merge?

---

## Zadanie 5 -- In teractive Rebase: porządkowanie historii przed PR

**Ćwiczone narzędzia:** `git rebase -i HEAD~N`, opcje `pick`/`reword`/`squash`/`fixup`/`drop`, push `--force-with-lease`.

**Opis:** Stworzysz "brudną" gałąź z 5 commitami (typo, WIP do usunięcia, fixup) i uporządkujesz ją w 3 czyste commity przed otwarciem PR.

### Przygotowanie

#### Lokalnie

```bash
mkdir git-zad5-rebase-interactive && cd git-zad5-rebase-interactive
git init
git branch -M main
```

#### Na GitHubie

1. `git-zad5-rebase-interactive`, Public, BEZ README.
2. Lokalnie:

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-zad5-rebase-interactive.git
```

### Skrypt setupowy

```bash
# Inicjalny commit na main
echo "# User Registration Feature" > README.md
git add README.md
git commit -m "init: empty project"
git push -u origin main

# Gałąź feature/user-registration -- 5 brudnych commitów
git switch -c feature/user-registration

# Commit 1
cat > User.java << 'EOF'
public class User {
    private String name;
    private String emial;    // literówka!
}
EOF
git add User.java
git commit -m "add user"

# Commit 2
cat > UserService.java << 'EOF'
public class UserService {
    public void register(User user) {}
}
EOF
git add UserService.java
git commit -m "service"

# Commit 3 -- fix typo (powinno być z commitem 1)
cat > User.java << 'EOF'
public class User {
    private String name;
    private String email;
}
EOF
git add User.java
git commit -m "fix typo in User"

# Commit 4 -- WIP do usunięcia
echo "TODO: dokończyć" > NOTES.txt
git add NOTES.txt
git commit -m "WIP do usuniecia"

# Commit 5 -- testy
cat > UserTest.java << 'EOF'
public class UserTest {
    public void testRegister() {}
}
EOF
git add UserTest.java
git commit -m "tests"

# Push brudnej historii jako BACKUP (żebyś miał punkt odniesienia)
git push -u origin feature/user-registration:feature/user-registration-original

# Push tej samej gałęzi pod swoją nazwą (będziemy ją czyścić)
git push -u origin feature/user-registration

git log --oneline
```

Po skrypcie masz 5 commitów: `add user`, `service`, `fix typo`, `WIP do usuniecia`, `tests`.

### Scenariusz -- Wariant A (CLI / Git Bash)

1. Wystartuj interactive rebase na ostatnich 5 commitach:

   ```bash
   git rebase -i HEAD~5
   ```

2. Otworzy się edytor (nano / vim / VS Code -- ten ustawiony w sekcji 0.4):

   ```
   pick abc1111 add user
   pick def2222 service
   pick ghi3333 fix typo in User
   pick jkl4444 WIP do usuniecia
   pick mno5555 tests
   ```

3. **Edytuj listę** -- przesuń `fix typo` pod `add user` jako `fixup`, usuń WIP, daj reword na trzy ważne commity:

   ```
   reword abc1111 add user
   fixup ghi3333 fix typo in User
   reword def2222 service
   drop jkl4444 WIP do usuniecia
   pick mno5555 tests
   ```

4. Zapisz i zamknij edytor.

5. Git otworzy edytor dla `reword abc1111` -- zmień wiadomość na:

   ```
   feat: add User entity with name and email
   ```

   Zapisz, zamknij.

6. Git otworzy edytor dla `reword def2222` -- zmień wiadomość na:

   ```
   feat: add UserService with registration
   ```

   Zapisz, zamknij.

7. Sprawdź:

   ```bash
   git log --oneline
   # nowy_h3 tests
   # nowy_h2 feat: add UserService with registration
   # nowy_h1 feat: add User entity with name and email
   ```

   Zamiast 5 commitów -- 3 czyste!

8. Push z `--force-with-lease` (bo historia się zmieniła):

   ```bash
   git push --force-with-lease
   ```

### Scenariusz -- Wariant B (IntelliJ IDEA)

> **Reset przed wariantem B:** w CLI `git reset --hard origin/feature/user-registration-original` (przywróć brudną historię).

1. *Git → Show Git Log* (`Alt+9`).
2. Znajdź **6. commit od końca** (czyli commit `init: empty project` -- on jest "punktem startu" rebase).

   > **Uwaga:** "Interactively Rebase from Here" rebase'uje commity **po** wybranym -- czyli zaznacz commit, **który ma zostać** (parent rebase'owanego zakresu).

3. Prawy klik na `init: empty project` → **Interactively Rebase from Here…**.
4. IntelliJ pokaże listę 5 commitów feature/user-registration z przyciskami **Pick / Reword / Fixup / Squash / Drop**.
5. Dla każdego commita:
    - `add user` → **Reword** (po klikniciu *Start Rebasing* IntelliJ poprosi o nową wiadomość: `feat: add User entity with name and email`).
    - `fix typo in User` → **Fixup** + przesuń (drag & drop) **pod** `add user`.
    - `service` → **Reword** (`feat: add UserService with registration`).
    - `WIP do usuniecia` → **Drop**.
    - `tests` → **Pick** (zostaje jak jest).
6. **Start Rebasing**.
7. *Git → Push* → strzałka obok Push → **Force Push**.

### Co ma być na GitHubie po zadaniu

* [ ] Repo `git-zad5-rebase-interactive` publiczne.
* [ ] Gałąź `feature/user-registration-original` -- **5 commitów** (brudna historia).
* [ ] Gałąź `feature/user-registration` -- **3 commity** (po rebase).
* [ ] Wszystkie commity z `feature/user-registration` mają **profesjonalne** wiadomości (`feat: ...`).

### Pytania (do RAPORT.md)

1. Jaka jest różnica między `squash` a `fixup`? (Wskazówka: co dzieje się z wiadomością commit?)
2. Dlaczego po `git rebase -i` musisz użyć `git push --force-with-lease`? Co by się stało gdybyś użył zwykłego `git push`?
3. Co zrobi się złego, jeśli zrobisz `rebase -i` na `main` współdzielonym z zespołem?
4. Załącz screenshot `git log --oneline` **przed** rebase i **po** rebase.
5. Co robi `drop` -- usuwa zmiany z plików czy tylko z historii? Sprawdź zawartość `NOTES.txt` po rebase.

---

## Zadanie 6 -- Reflog: odzyskanie po `reset --hard`

**Ćwiczone narzędzia:** `git reflog`, `git reset --hard HEAD@{N}`, odzyskiwanie usuniętej gałęzi.

**Opis:** Celowo "stracisz" 2 commity przez `git reset --hard HEAD~2` i odzyskasz je przez reflog. Bonus: usuniesz gałąź i ją odzyskasz.

### Przygotowanie

#### Lokalnie

```bash
mkdir git-zad6-reflog && cd git-zad6-reflog
git init
git branch -M main
```

#### Na GitHubie

1. `git-zad6-reflog`, Public, BEZ README.
2. Lokalnie:

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-zad6-reflog.git
```

### Skrypt setupowy

```bash
echo "Notatka A" > notes.txt
git add notes.txt
git commit -m "Commit A"

echo "Notatka B" >> notes.txt
git add notes.txt
git commit -m "Commit B"

echo "Notatka C" >> notes.txt
git add notes.txt
git commit -m "Commit C"

git log --oneline
git push -u origin main
```

### Scenariusz -- Wariant A (CLI / Git Bash)

1. Sprawdź, że masz 3 commity:

   ```bash
   git log --oneline
   # hashC Commit C
   # hashB Commit B
   # hashA Commit A
   ```

2. **CELOWA POMYŁKA** -- usuwamy 2 ostatnie commity:

   ```bash
   git reset --hard HEAD~2
   git log --oneline
   # hashA Commit A          ← Commit B i C zniknęły!
   cat notes.txt              # tylko "Notatka A"
   ```

3. **PANIKA?** Nie -- reflog pamięta wszystko:

   ```bash
   git reflog
   # hashA HEAD@{0}: reset: moving to HEAD~2
   # hashC HEAD@{1}: commit: Commit C       ← TO chcemy odzyskać
   # hashB HEAD@{2}: commit: Commit B
   # hashA HEAD@{3}: commit: Commit A
   # ...
   ```

4. **Odzyskaj** stan sprzed `reset`:

   ```bash
   git reset --hard HEAD@{1}
   git log --oneline
   # hashC Commit C            ← przywrócone!
   # hashB Commit B            ← przywrócone!
   # hashA Commit A
   cat notes.txt              # wszystkie 3 notatki
   ```

5. Push (jeśli wcześniej pushowałeś, możesz potrzebować `--force-with-lease`, ale tu nie zmieniliśmy nic na remote):

   ```bash
   git push
   ```

6. **Bonus -- odzyskanie usuniętej gałęzi:**

   ```bash
   git switch -c feature/eksperyment
   echo "Eksperyment" > experiment.txt
   git add experiment.txt
   git commit -m "feat: eksperyment"
   git switch main
   git branch -D feature/eksperyment    # usunięcie gałęzi (siłowo)

   git log feature/eksperyment           # error: gałąź nie istnieje
   git reflog                            # ale commit "feat: eksperyment" jest w reflog!
   # zanotuj hash commita "feat: eksperyment"
   git switch -c feature/eksperyment-recovered <hash>
   git log --oneline                     # commit jest z powrotem!
   ```

### Scenariusz -- Wariant B (IntelliJ IDEA)

> **IntelliJ nie ma natywnego widoku reflog.** Używaj wbudowanego Terminala.

1. Otwórz katalog w IntelliJ.
2. *Git → Show Git Log*. Znajdź `Commit C` (najnowszy). Prawy klik → **Reset Current Branch to Here…** → ❌ -- to nie jest to czego chcemy. Zamiast tego znajdź `Commit A` i zaznacz tryb **Hard**:
    - Prawy klik na `Commit A` → **Reset Current Branch to Here…** → wybierz **Hard** → **Reset**.
3. Git Log pokazuje teraz tylko `Commit A`. **PANIKA**.
4. Otwórz Terminal IntelliJ (`Alt+F12`):

   ```bash
   git reflog
   ```

   Zanotuj hash z linii `HEAD@{1}` (przed reset).

5. Wróć do Git Log. **Hash z reflog** wpisz w pole filtru w lewym górnym rogu Log -- IntelliJ pokaże ten commit. Prawy klik → **Reset Current Branch to Here…** → **Hard** → **Reset**.
6. `Commit B` i `Commit C` są z powrotem!

### Co ma być na GitHubie po zadaniu

* [ ] Repo `git-zad6-reflog` publiczne, gałąź `main` z 3 commitami (A, B, C).
* [ ] (Opcjonalnie z bonusu) Gałąź `feature/eksperyment-recovered` z odzyskanym commitem.
* [ ] W RAPORT.md zrzut ekranu `git reflog` z zaznaczonym wpisem `reset` i wpisem przed nim.

### Pytania (do RAPORT.md)

1. Jak długo żyją wpisy reflog? (Wskazówka: zobacz `git config gc.reflogExpire`.)
2. Czy reflog idzie na remote po `git push`? Sprawdź: zaloguj się na GitHubie i sprawdź, czy widzisz tam reflog.
3. Jak rozpoznać w `git reflog`, który `HEAD@{N}` to "przed pomyłkowym resetem"?
4. Czy reflog uratuje Cię, gdy: (a) przypadkowo `rm -rf` na całym katalogu repo? (b) przypadkowo `git reset --hard` ale jeszcze przed pushem? (c) usunąłeś gałąź lokalnie, ale była tylko lokalnie?
5. Załącz screenshot `git reflog` po reset i po recovery.

---

## Zadanie 7 -- Bisect: binarne polowanie na buga

**Ćwiczone narzędzia:** `git bisect start`, `git bisect bad`, `git bisect good <hash>`, `git bisect reset`, opcjonalnie `git bisect run <skrypt>`.

**Opis:** W repo z 8 commitami `Calc.java` jest jeden, który wprowadził buga w metodzie `multiply` (`a + b` zamiast `a * b`). Znajdź go używając binary search.

### Przygotowanie

#### Lokalnie

```bash
mkdir git-zad7-bisect && cd git-zad7-bisect
git init
git branch -M main
```

#### Na GitHubie

1. `git-zad7-bisect`, Public, BEZ README.
2. Lokalnie:

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-zad7-bisect.git
```

### Skrypt setupowy (długi -- skopiuj **całość**)

```bash
# === COMMIT 1: szkielet ===
cat > Calc.java << 'EOF'
public class Calc {
    public static void main(String[] args) {
        System.out.println("=== Calculator ===");
    }
}
EOF
git add Calc.java
git commit -m "feat: szkielet kalkulatora"

# === COMMIT 2: add ===
cat > Calc.java << 'EOF'
public class Calc {
    public static int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) {
        System.out.println("=== Calculator ===");
        System.out.println("add(2,3) = " + add(2, 3));
    }
}
EOF
git add Calc.java
git commit -m "feat: add method"

# === COMMIT 3: subtract ===
cat > Calc.java << 'EOF'
public class Calc {
    public static int add(int a, int b) {
        return a + b;
    }
    public static int subtract(int a, int b) {
        return a - b;
    }

    public static void main(String[] args) {
        System.out.println("=== Calculator ===");
        System.out.println("add(2,3) = " + add(2, 3));
        System.out.println("subtract(5,2) = " + subtract(5, 2));
    }
}
EOF
git add Calc.java
git commit -m "feat: subtract method"

# === COMMIT 4: multiply Z BUGIEM ===
cat > Calc.java << 'EOF'
public class Calc {
    public static int add(int a, int b) {
        return a + b;
    }
    public static int subtract(int a, int b) {
        return a - b;
    }
    public static int multiply(int a, int b) {
        return a + b;   // BUG! powinno być a * b
    }

    public static void main(String[] args) {
        System.out.println("=== Calculator ===");
        System.out.println("add(2,3) = " + add(2, 3));
        System.out.println("subtract(5,2) = " + subtract(5, 2));
        System.out.println("multiply(2,3) = " + multiply(2, 3));
    }
}
EOF
git add Calc.java
git commit -m "feat: multiply method"

# === COMMIT 5: divide ===
cat > Calc.java << 'EOF'
public class Calc {
    public static int add(int a, int b) {
        return a + b;
    }
    public static int subtract(int a, int b) {
        return a - b;
    }
    public static int multiply(int a, int b) {
        return a + b;   // BUG NADAL TU
    }
    public static int divide(int a, int b) {
        return a / b;
    }

    public static void main(String[] args) {
        System.out.println("=== Calculator ===");
        System.out.println("add(2,3) = " + add(2, 3));
        System.out.println("subtract(5,2) = " + subtract(5, 2));
        System.out.println("multiply(2,3) = " + multiply(2, 3));
        System.out.println("divide(6,2) = " + divide(6, 2));
    }
}
EOF
git add Calc.java
git commit -m "feat: divide method"

# === COMMIT 6: komentarze ===
cat > Calc.java << 'EOF'
public class Calc {
    // Basic arithmetic operations
    public static int add(int a, int b) {
        return a + b;
    }
    public static int subtract(int a, int b) {
        return a - b;
    }
    public static int multiply(int a, int b) {
        return a + b;   // BUG NADAL TU
    }
    public static int divide(int a, int b) {
        return a / b;
    }

    public static void main(String[] args) {
        System.out.println("=== Calculator ===");
        System.out.println("add(2,3) = " + add(2, 3));
        System.out.println("subtract(5,2) = " + subtract(5, 2));
        System.out.println("multiply(2,3) = " + multiply(2, 3));
        System.out.println("divide(6,2) = " + divide(6, 2));
    }
}
EOF
git add Calc.java
git commit -m "docs: add comments"

# === COMMIT 7: refactor pomocniczy ===
cat > Calc.java << 'EOF'
public class Calc {
    // Basic arithmetic operations
    public static int add(int a, int b) {
        return a + b;
    }
    public static int subtract(int a, int b) {
        return a - b;
    }
    public static int multiply(int a, int b) {
        return a + b;   // BUG NADAL TU
    }
    public static int divide(int a, int b) {
        return a / b;
    }

    private static String format(String op, int a, int b, int result) {
        return op + "(" + a + "," + b + ") = " + result;
    }

    public static void main(String[] args) {
        System.out.println("=== Calculator ===");
        System.out.println(format("add", 2, 3, add(2, 3)));
        System.out.println(format("subtract", 5, 2, subtract(5, 2)));
        System.out.println(format("multiply", 2, 3, multiply(2, 3)));
        System.out.println(format("divide", 6, 2, divide(6, 2)));
    }
}
EOF
git add Calc.java
git commit -m "refactor: extract format helper"

# === COMMIT 8: extra demo ===
cat > Calc.java << 'EOF'
public class Calc {
    // Basic arithmetic operations
    public static int add(int a, int b) {
        return a + b;
    }
    public static int subtract(int a, int b) {
        return a - b;
    }
    public static int multiply(int a, int b) {
        return a + b;   // BUG NADAL TU
    }
    public static int divide(int a, int b) {
        return a / b;
    }

    private static String format(String op, int a, int b, int result) {
        return op + "(" + a + "," + b + ") = " + result;
    }

    public static void main(String[] args) {
        System.out.println("=== Calculator ===");
        System.out.println(format("add", 2, 3, add(2, 3)));
        System.out.println(format("subtract", 5, 2, subtract(5, 2)));
        System.out.println(format("multiply", 2, 3, multiply(2, 3)));
        System.out.println(format("divide", 6, 2, divide(6, 2)));
        System.out.println(format("multiply", 4, 5, multiply(4, 5)));
    }
}
EOF
git add Calc.java
git commit -m "feat: dodatkowy przykład multiply"

git push -u origin main
git log --oneline
```

Zanotuj **hash pierwszego commita** ("szkielet kalkulatora") -- będzie potrzebny.

### Scenariusz -- Wariant A (CLI / Git Bash)

1. Sprawdź, że bug jest:

   ```bash
   javac Calc.java && java Calc
   ```

   Powinno pokazać `multiply(2,3) = 5` (powinno być 6).

2. Wystartuj bisect:

   ```bash
   git bisect start
   git bisect bad                                        # HEAD jest zły
   git bisect good <hash-pierwszego-commita>             # zanotowany wyżej
   ```

   Git przełączy się na środkowy commit i wypisze np.:
   ```
   Bisecting: 3 revisions left to test after this (roughly 2 steps)
   ```

3. **W każdej iteracji** -- skompiluj, uruchom, oceń:

   ```bash
   javac Calc.java && java Calc
   ```

   Sprawdź wynik `multiply(2,3)`:
    - Jeśli **5** → `git bisect bad`.
    - Jeśli **6** lub `multiply` jeszcze nie istnieje w pliku → `git bisect good` (patrz "Uwaga" niżej).

   > **Uwaga:** w commitach 1-3 metody `multiply` jeszcze nie ma -- traktuj te commity jako `good`, bo nie zawierają bug-a (bug pojawia się dopiero z dodaniem `multiply`).

   Powtarzaj `git bisect good` / `git bisect bad`, aż Git wypisze:
   ```
   <hash> is the first bad commit
   commit <hash>
   ...
       feat: multiply method
   ```

4. Zakończ bisect:

   ```bash
   git bisect reset
   ```

   Sprawdź, co bisect faktycznie znalazł:

   ```bash
   git show <hash-z-bisecta>
   ```

   W diffie powinieneś zobaczyć **dodanie** metody `multiply(int a, int b) { return a + b; }` -- to jest commit wprowadzający buga (`feat: multiply method`).

> **Pułapka -- „bisect wskazał inny commit niż blame / Annotate":** Po zakończeniu bisecta odruchowo używasz `git blame Calc.java` (albo w IntelliJ: prawym przyciskiem na rynnie → *Annotate with Git Blame*) i widzisz, że feralna linia jest przypisana do **commita 5 albo późniejszego**, a nie do tego z bisecta. To nie jest błąd -- to różnica między narzędziami:
>
> - `git bisect` szuka **pierwszego commita, w którym test failuje** -- tu commit 4 (`feat: multiply method`), bo dopiero od niego `multiply(2,3)` zwraca 5.
> - `git blame` / *Annotate* pokazuje **ostatni commit, który dotknął danej linii**. W setupie zadania commit 5 zmienia komentarz w linii z bugiem (`// BUG! powinno być a * b` → `// BUG NADAL TU`), więc dla Gita linia została „zmodyfikowana" -- mimo że kod (`return a + b`) jest identyczny. Blame przypisuje ją commitowi 5.
>
> Zasada: do szukania **regresji behawioralnej** („od kiedy program się zepsuł?") używaj `git bisect`. Do szukania **autorstwa konkretnej linii** -- `git blame`. To dwa różne pytania i często dają dwie różne odpowiedzi.

5. **Bonus -- automatyczny bisect przez skrypt:**

   Utwórz `test.sh`:

   ```bash
   cat > test.sh << 'EOF'
   #!/bin/bash
   javac Calc.java 2>/dev/null || exit 125          # commit nie kompiluje się -- skip
   OUT=$(java Calc 2>/dev/null)
   if echo "$OUT" | grep -q "multiply(2,3) = 6"; then
       exit 0   # good
   elif echo "$OUT" | grep -q "multiply"; then
       exit 1   # bad
   else
       exit 0   # multiply nie istnieje -- też good
   fi
   EOF
   chmod +x test.sh

   git bisect start
   git bisect bad
   git bisect good <hash-pierwszego-commita>
   git bisect run ./test.sh
   git bisect reset
   ```

   Git automatycznie przejdzie wszystkie kroki i wskaże winowajcę.

### Scenariusz -- Wariant B (IntelliJ IDEA)

> **IntelliJ nie ma natywnego widoku bisect.** Używaj wbudowanego Terminala (`Alt+F12`) i komend CLI z wariantu A. *Git → Log* podświetli aktualnie sprawdzany commit.

1. Otwórz katalog w IntelliJ.
2. *Git → Show Git Log* (`Alt+9`) -- zobacz wszystkie 8 commitów.
3. Otwórz Terminal (`Alt+F12`) i wykonaj kroki 1-4 wariantu A. Po każdym `git bisect good`/`bad` Log automatycznie pokazuje, gdzie jesteś.
4. Po zakończeniu (`is the first bad commit`) możesz w *Git → Log* prawym kliknąć na zły commit → **Show Diff** żeby zobaczyć, co dokładnie wprowadziło buga.

### Co ma być na GitHubie po zadaniu

* [ ] Repo `git-zad7-bisect` publiczne z 8 commitami.
* [ ] W RAPORT.md: hash złego commita, ile kroków bisect potrzebował, screenshot końcowego komunikatu `is the first bad commit`.

### Pytania (do RAPORT.md)

1. Ile kroków potrzebował bisect, żeby znaleźć buga? Jak ta liczba ma się do `log₂(8) = 3`?
2. Hash i wiadomość złego commita -- skopiuj z konsoli.
3. Kiedy bisect jest lepszy niż `git log` + `git diff`?
4. Co robi `git bisect run <skrypt>`? Jakie kody wyjścia ma rozumieć skrypt?
5. Co zrobić, jeśli któryś commit się **nie kompiluje** podczas bisect i nie da się ocenić? (Wskazówka: `git bisect skip` + exit 125 w `bisect run`.)

---

## Zadanie 8 -- Reset (3 tryby) vs Revert

**Ćwiczone narzędzia:** `git reset --soft`, `git reset --mixed` (domyślny), `git reset --hard`, `git revert`.

**Opis:** Stworzysz 3 commity i przećwiczysz 3 tryby reset (każdy na osobnej gałęzi -- żeby porównać efekty). Następnie zrobisz `revert` na pushowanym commit.

### Przygotowanie

#### Lokalnie

```bash
mkdir git-zad8-reset-revert && cd git-zad8-reset-revert
git init
git branch -M main
```

#### Na GitHubie

1. `git-zad8-reset-revert`, Public, BEZ README.
2. Lokalnie:

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-zad8-reset-revert.git
```

### Skrypt setupowy

```bash
echo "wersja A" > file.txt
git add file.txt
git commit -m "Commit A"

echo "wersja B" > file.txt
git add file.txt
git commit -m "Commit B"

echo "wersja C" > file.txt
git add file.txt
git commit -m "Commit C"

git push -u origin main

# 3 gałęzie testowe -- każda do innego trybu reset
git branch test/soft
git branch test/mixed
git branch test/hard

git log --oneline
```

### Scenariusz -- Wariant A (CLI / Git Bash)

#### Część 1 -- `--soft` (zachowuje zmiany w staging)

```bash
git switch test/soft
git reset --soft HEAD~1            # cofa Commit C, ale zmiany trafiają do staging
git log --oneline                  # widzisz tylko A i B
git status
# Changes to be committed:
#   modified: file.txt              ← zawartość "wersja C" w staging!

cat file.txt                       # nadal "wersja C" -- bo plik nie został wycofany

git commit -m "Commit C poprawiony"
git log --oneline                  # nowy hash dla "Commit C poprawiony"
```

#### Część 2 -- `--mixed` (domyślny, zachowuje zmiany w working dir)

```bash
git switch test/mixed
git reset HEAD~1                   # bez flagi = --mixed
git log --oneline                  # tylko A i B
git status
# Changes not staged for commit:
#   modified: file.txt              ← zmiany w working dir, NIE w staging

cat file.txt                       # "wersja C"

git add file.txt
git commit -m "Commit C poprawiony"
```

#### Część 3 -- `--hard` (USUWA zmiany!)

```bash
git switch test/hard
git reset --hard HEAD~1
git log --oneline                  # tylko A i B
git status                         # nothing to commit, working tree clean
cat file.txt                       # "wersja B" -- WERSJA C ZNIKNĘŁA!

# Ratunek przez reflog:
git reflog
# Znajdź wpis "commit: Commit C" (HEAD@{N})
git reset --hard HEAD@{1}          # przywróć
cat file.txt                       # "wersja C" -- z powrotem!
```

#### Część 4 -- `git revert` (bezpieczne cofnięcie pushowanego commita)

```bash
git switch main
git push                            # upewnij się że main jest aktualny
git revert HEAD                     # otworzy się edytor z wiadomością "Revert ..."
# zapisz i zamknij edytor
git log --oneline
# new_hash Revert "Commit C"        ← nowy commit cofający
# hash3 Commit C                    ← oryginalny nadal w historii
# hash2 Commit B
# hash1 Commit A

git push                            # bez --force-with-lease, bo to nowy commit, nie zmiana historii
```

#### Część 5 -- push gałęzi testowych

```bash
git push origin test/soft test/mixed test/hard
```

### Scenariusz -- Wariant B (IntelliJ IDEA)

#### Część 1-3 -- 3 tryby reset

Dla każdej z gałęzi `test/soft`, `test/mixed`, `test/hard`:

1. Branches → checkout odpowiedniej gałęzi.
2. *Git → Show Git Log* (`Alt+9`).
3. Prawy klik na **Commit B** (commit, do którego cofamy) → **Reset Current Branch to Here…**.
4. W dialogu wybierz **Soft** / **Mixed** / **Hard** → **Reset**.
5. *Git → Commit* (`Ctrl+K`) (gdy `Soft` lub `Mixed`) → wpisz wiadomość → Commit.

Dla `test/hard` po Hard reset użyj Terminala (`Alt+F12`):

```bash
git reflog
git reset --hard HEAD@{N}           # gdzie N to wpis przed reset
```

#### Część 4 -- Revert

1. Branches → checkout `main`.
2. *Git → Show Git Log* → prawy klik na **Commit C** → **Revert Commit**.
3. IntelliJ otworzy dialog commit z wiadomością `Revert "Commit C"`. Kliknij **Commit**.
4. *Git → Push*.

### Co ma być na GitHubie po zadaniu

* [ ] Repo `git-zad8-reset-revert` publiczne.
* [ ] Gałęzie: `main` (4 commity: A, B, C, Revert "Commit C"), `test/soft`, `test/mixed`, `test/hard` (każda po 3 commity, z trzecim zrekonstruowanym).
* [ ] W RAPORT.md tabela różnic 3 trybów reset.

### Pytania (do RAPORT.md)

1. Wypełnij tabelę:

| Tryb | `git status` po reset | Zawartość pliku w working dir | Czy zmiany w staging? |
|------|-----------------------|------------------------------|----------------------|
| `--soft` | ? | ? | ? |
| `--mixed` | ? | ? | ? |
| `--hard` | ? | ? | ? |

2. Dlaczego `git revert` jest bezpieczny dla zespołu, a `git reset --hard` na pushowanym branchu **nie**?
3. Po `git reset --hard` -- czy stracone commity są usunięte z dysku? Jak je odzyskać?
4. Kiedy chcesz zrobić `--soft` zamiast `--mixed`?
5. Załącz screenshot `git log --oneline` po `git revert HEAD` -- czy oryginalny `Commit C` jest jeszcze w historii?

---

## Zadanie 9 -- Feature branch + PR z konfliktem (GitHub flow)

**Ćwiczone narzędzia:** `git fetch origin`, `git rebase origin/main`, otwieranie PR przez GitHub UI, "Squash and merge" vs "Create a merge commit", rozwiązywanie konfliktu w trakcie rebase'a.

**Opis:** Symulujesz pracę dwuosobową w jednym repozytorium. Ty pracujesz lokalnie na gałęzi `feature/dodaj-todo`, "kolega" w międzyczasie edytuje `main` bezpośrednio przez GitHub UI (commit z poziomu przeglądarki). Lokalnie aktualizujesz feature, rozwiązujesz konflikt, otwierasz PR z `feature/dodaj-todo` do `main` i mergujesz.

> **Uwaga -- dlaczego nie fork?** Klasyczny "fork + PR z forka do upstream" wymaga relacji fork-upstream w bazie GitHuba, którą zakłada wyłącznie przycisk **Fork**. GitHub nie pozwala fork-ować własnego repo do tego samego konta, więc dwa ręcznie utworzone repo (klon + przepięcie remote'ów) **nie są** dla GitHuba fork-iem -- żółty banner "Compare & pull request" się nie pojawi, a w *New pull request* nie da się wybrać drugiego repo. Z tego powodu w tym zadaniu używamy układu **single-repo / feature branch**, który dokładnie odwzorowuje codzienny GitHub flow w firmie. Jeśli chcesz w przyszłości przećwiczyć prawdziwy fork-workflow, sforkuj dowolne publiczne repo z drugiego konta lub z konta organizacji.

### Przygotowanie

#### Lokalnie

```bash
mkdir git-zad9 && cd git-zad9
git init
git branch -M main
```

#### Na GitHubie

1. [https://github.com/new](https://github.com/new) → `git-zad9`, Public, BEZ README.
2. Lokalnie:

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-zad9.git
```

### Skrypt setupowy

```bash
cat > README.md << 'EOF'
# Projekt: lista TODO

- [ ] Zadanie 1: setup projektu
- [ ] Zadanie 2: napisz README
- [ ] Zadanie 3: dodaj testy
- [ ] Zadanie 4: wdróż
- [ ] Zadanie 5: dokumentacja
EOF

cat > CHANGELOG.md << 'EOF'
# Changelog
EOF

git add README.md CHANGELOG.md
git commit -m "init: TODO list + changelog"
git push -u origin main
```

Sprawdź:

```bash
git remote -v
# origin    https://github.com/<TWOJ-USER>/git-zad9.git (fetch/push)
```

### Scenariusz -- Wariant A (CLI / Git Bash)

#### Krok 1 -- twoja praca na feature branch

```bash
# Jesteś w git-zad9
git switch -c feature/dodaj-todo

# Edytuj linię 3 (Zadanie 1)
sed -i.bak 's|Zadanie 1: setup projektu|Zadanie 1: setup projektu (zrobione przez Ciebie)|' README.md && rm README.md.bak

git add README.md
git commit -m "feat: oznacz zadanie 1 jako zrobione"
git push -u origin feature/dodaj-todo
```

#### Krok 2 -- (opcjonalnie) otwórz PR teraz, żeby zobaczyć stan "no conflicts"

1. Otwórz `https://github.com/<TWOJ-USER>/git-zad9` -- powinien się pojawić **żółty banner** *"feature/dodaj-todo had recent pushes"* z przyciskiem **Compare & pull request**.
2. Kliknij banner → *base: main, compare: feature/dodaj-todo* → **Create pull request** → tytuł i opis → **Create**.
3. PR jest "green" -- "Able to merge". Zostaw go otwartego, wrócimy do niego w Kroku 5.

> Jeśli wolisz, możesz pominąć Krok 2 i otworzyć PR dopiero po rozwiązaniu konfliktu (Krok 5). Ale zrobienie PR teraz pozwoli Ci za chwilę zobaczyć w UI komunikat **"This branch has conflicts"** -- materiał na screenshot do RAPORT.md.

#### Krok 3 -- "kolega" zmienia tę samą linię w `main` przez GitHub UI

W przeglądarce otwórz `https://github.com/<TWOJ-USER>/git-zad9/blob/main/README.md` → przycisk **edit** (ołówek) → zmień linię 3 na:

```
- [ ] Zadanie 1: setup projektu (zrobione przez kolegę)
```

→ scroll do dołu → **Commit changes…** → Commit message: `feat: opis setupu`. **Commit directly to the main branch** → **Commit changes**.

> Jeśli masz otwarty PR z Kroku 2, odśwież go teraz -- zobaczysz czerwony komunikat **"This branch has conflicts that must be resolved"**. Zrób screenshot do RAPORT.md.

#### Krok 4 -- synchronizacja feature brancha i konflikt

```bash
git fetch origin
git switch feature/dodaj-todo
git rebase origin/main
```

Git wypisze konflikt w `README.md`. Edytuj plik, wybierz **swoją** wersję, usuń markery `<<<<<<<`, `=======`, `>>>>>>>`:

```
- [ ] Zadanie 1: setup projektu (zrobione przez Ciebie)
```

```bash
git add README.md
git rebase --continue
git push --force-with-lease
```

#### Krok 5 -- domknij PR

1. Jeśli PR był otwarty od Kroku 2: odśwież stronę -- konflikt powinien zniknąć, PR znów jest "green".
   Jeśli nie otwierałeś PR wcześniej: wróć na `https://github.com/<TWOJ-USER>/git-zad9`, żółty banner powinien być nadal widoczny (force-push z Kroku 4 jest świeży) → **Compare & pull request** → **Create pull request**.
2. Kliknij **Squash and merge** → **Confirm squash and merge** → **Delete branch** (opcjonalnie).

> **Eksperyment:** zrób drugi PR z innej gałęzi (np. dodaj zmianę w CHANGELOG.md), tym razem zmerguj przez **Create a merge commit**. Porównaj historię w `main` (`git log --oneline --graph` po `git pull`) -- po squash merge widzisz 1 commit, po merge commit widzisz 2 commity + commit merge.

### Scenariusz -- Wariant B (IntelliJ IDEA)

1. Otwórz katalog `git-zad9` w IntelliJ.
2. Branches → **New Branch from main** → `feature/dodaj-todo`.
3. Edytuj `README.md` → *Git → Commit* (`Ctrl+K`) → wiadomość → **Commit and Push**.
4. (Opcjonalnie) otwórz PR w przeglądarce -- zobaczysz "Able to merge".
5. **Symulacja zmiany w main:** zrób tę zmianę w przeglądarce (jak w wariancie A Krok 3) -- commit prosto na `main`.
6. *VCS → Update Project* (`Ctrl+T`) -- fetch.
7. *Git → Rebase Current Branch onto…* → wybierz `origin/main` → konflikt → 3-panel merge → Apply.
8. *Git → Push* → **Force Push** (force-with-lease).
9. Wróć do PR-a w przeglądarce → **Squash and merge**.

### Co ma być na GitHubie po zadaniu

* [ ] Repo `git-zad9` z gałęzią `main` i (zmergowaną przez squash) gałęzią `feature/dodaj-todo`.
* [ ] PR z `feature/dodaj-todo` do `main` **zmergowany** (Squash and merge).
* [ ] (Bonus) Drugi PR zmergowany przez "Create a merge commit" -- żeby porównać historię.
* [ ] W RAPORT.md zrzut ekranu PR z "This branch has conflicts" (z Kroku 3) **lub** zrzut zmergowanego PR.

### Pytania (do RAPORT.md)

1. Po co w ogóle pracować na osobnej gałęzi `feature/...` zamiast commitować bezpośrednio na `main`?
2. Kiedy używasz `git fetch origin` + `git merge origin/main`, a kiedy `git fetch origin` + `git rebase origin/main`? Jak to wpływa na historię?
3. Jaka jest różnica między **Squash and merge**, **Create a merge commit** i **Rebase and merge** w UI GitHuba?
4. Dlaczego po lokalnym `git rebase origin/main` musisz pushować `--force-with-lease` (zamiast zwykłego `--force`)?
5. Wymień 3 dobre praktyki przy zgłaszaniu PR (commit messages, opis, rozmiar PR, code review).

---

## Zadanie 10 -- Śledztwo: kombinacja technik (samodzielne)

**Ćwiczone narzędzia:** **wszystkie** poznane wcześniej -- sam dobierz właściwe.

**Opis:** Skopiuj skrypt setupowy. Stworzy on repo z **3 problemami** ukrytymi w różnych miejscach. Twoje zadanie: znajdź każdy problem, dobierz właściwe narzędzie Git i go rozwiąż. Zapisz w RAPORT.md: **jakie narzędzie**, **co zaobserwowałeś**, **jak rozwiązałeś**.

### Przygotowanie

#### Lokalnie

```bash
mkdir git-zad10-sledztwo && cd git-zad10-sledztwo
git init
git branch -M main
```

#### Na GitHubie

1. `git-zad10-sledztwo`, Public, BEZ README.
2. Lokalnie:

```bash
git remote add origin https://github.com/<TWOJ-USER>/git-zad10-sledztwo.git
```

### Skrypt setupowy (skopiuj i wklej **całość** -- jeden raz!)

```bash
# === Problem (a): "stracone" commity ===
echo "Notatka 1" > notes.txt
git add notes.txt
git commit -m "Commit 1"

echo "Notatka 2" >> notes.txt
git add notes.txt
git commit -m "Commit 2"

echo "Notatka 3" >> notes.txt
git add notes.txt
git commit -m "Commit 3"

echo "Notatka 4" >> notes.txt
git add notes.txt
git commit -m "Commit 4"

# CELOWE pomyłkowe cofnięcie 2 commitów -- problem (a) gotowy
git reset --hard HEAD~2

# === Problem (b): bug w jednym z 8 commitów Greeting.java ===
cat > Greeting.java << 'EOF'
public class Greeting {
    public static String greet(String name) {
        return "Hello, " + name + "!";
    }
}
EOF
git add Greeting.java
git commit -m "feat: dodaj klasę Greeting (commit B1)"

cat > Greeting.java << 'EOF'
public class Greeting {
    public static String greet(String name) {
        return "Hello, " + name + "!";
    }

    public static String farewell(String name) {
        return "Goodbye, " + name + "!";
    }
}
EOF
git add Greeting.java
git commit -m "feat: dodaj farewell (commit B2)"

cat > Greeting.java << 'EOF'
public class Greeting {
    public static String greet(String name) {
        return "Hello, " + name + "!";
    }

    public static String farewell(String name) {
        return "Goodbye, " + name + "!";
    }

    public static String time(int hour) {
        if (hour < 12) return "morning";
        if (hour < 18) return "afternoon";
        return "evening";
    }
}
EOF
git add Greeting.java
git commit -m "feat: dodaj time (commit B3)"

cat > Greeting.java << 'EOF'
public class Greeting {
    public static String greet(String name) {
        return "Hello, " + name + "!";
    }

    public static String farewell(String name) {
        return "Goodbye, " + name + "!";
    }

    public static String time(int hour) {
        if (hour < 12) return "morning";
        if (hour < 18) return "afternoon";
        return "evening";
    }

    public static String constant() {
        return "PI=3.14";
    }
}
EOF
git add Greeting.java
git commit -m "feat: dodaj constant (commit B4)"

# Commit z BUGIEM -- "PI" zostaje błędnie zmienione na "Pi" (literówka w stałej)
cat > Greeting.java << 'EOF'
public class Greeting {
    public static String greet(String name) {
        return "Hello, " + name + "!";
    }

    public static String farewell(String name) {
        return "Goodbye, " + name + "!";
    }

    public static String time(int hour) {
        if (hour < 12) return "morning";
        if (hour < 18) return "afternoon";
        return "evening";
    }

    public static String constant() {
        return "Pi=3.14";   // BUG! mała "i"
    }

    public static int absoluteValue(int x) {
        return x < 0 ? -x : x;
    }
}
EOF
git add Greeting.java
git commit -m "feat: dodaj absoluteValue (commit B5)"

# Kolejne 3 commity -- bug zostaje
cat > Greeting.java << 'EOF'
public class Greeting {
    public static String greet(String name) {
        return "Hello, " + name + "!";
    }

    public static String farewell(String name) {
        return "Goodbye, " + name + "!";
    }

    public static String time(int hour) {
        if (hour < 12) return "morning";
        if (hour < 18) return "afternoon";
        return "evening";
    }

    public static String constant() {
        return "Pi=3.14";
    }

    public static int absoluteValue(int x) {
        return x < 0 ? -x : x;
    }

    public static int square(int x) {
        return x * x;
    }
}
EOF
git add Greeting.java
git commit -m "feat: dodaj square (commit B6)"

cat > Greeting.java << 'EOF'
public class Greeting {
    // Klasa pomocnicza z różnymi metodami narzędziowymi.
    public static String greet(String name) {
        return "Hello, " + name + "!";
    }

    public static String farewell(String name) {
        return "Goodbye, " + name + "!";
    }

    public static String time(int hour) {
        if (hour < 12) return "morning";
        if (hour < 18) return "afternoon";
        return "evening";
    }

    public static String constant() {
        return "Pi=3.14";
    }

    public static int absoluteValue(int x) {
        return x < 0 ? -x : x;
    }

    public static int square(int x) {
        return x * x;
    }
}
EOF
git add Greeting.java
git commit -m "docs: dodaj komentarz (commit B7)"

cat > Greeting.java << 'EOF'
public class Greeting {
    // Klasa pomocnicza z różnymi metodami narzędziowymi.
    public static String greet(String name) {
        return "Hello, " + name + "!";
    }

    public static String farewell(String name) {
        return "Goodbye, " + name + "!";
    }

    public static String time(int hour) {
        if (hour < 12) return "morning";
        if (hour < 18) return "afternoon";
        return "evening";
    }

    public static String constant() {
        return "Pi=3.14";
    }

    public static int absoluteValue(int x) {
        return x < 0 ? -x : x;
    }

    public static int square(int x) {
        return x * x;
    }

    public static int cube(int x) {
        return x * x * x;
    }
}
EOF
git add Greeting.java
git commit -m "feat: dodaj cube (commit B8)"

# === Problem (c): niedokończone zmiany w working dir + gałąź kolegi ===
git switch -c colleague/finished-feature
echo "Feature kolegi -- gotowe!" > colleague.txt
git add colleague.txt
git commit -m "feat: feature kolegi"

git switch main
echo "Moja niedokończona praca..." > wip.txt   # NIE commitujemy -- zostaje w working dir

git push -u origin main
git push -u origin colleague/finished-feature

echo ""
echo "=========================================="
echo "GOTOWE! Masz 3 problemy do rozwiązania:"
echo "(a) Brakuje 2 commitów na main -- gdzie one są?"
echo "(b) W historii Greeting.java jest bug 'Pi=3.14' (powinno być 'PI=3.14')"
echo "    -- znajdź który z 8 commitów wprowadził literówkę."
echo "(c) Masz niedokończony plik wip.txt; chcesz zmergować colleague/finished-feature."
echo "=========================================="
```

### Wskazówki (bez gotowych komend!)

**Problem (a):** Sprawdź `git log --oneline` -- masz tylko 2 commity (1 i 2), ale wcześniej tworzyłeś 4 commity. Co się stało? Pomyśl, jakie polecenie pokazuje **wszystkie** operacje HEAD, nie tylko te z aktualnej historii.

**Problem (b):** W pliku `Greeting.java` w `constant()` zwraca `"Pi=3.14"` zamiast `"PI=3.14"`. Jest 8 commitów na gałęzi po commitach problemu (a). Nie wiesz, który wprowadził buga -- ręcznie sprawdzanie 8 commitów to dużo. Czy znasz polecenie, które potrafi binarnie szukać buga w historii?

**Problem (c):** Masz `wip.txt` w working dir (niezacommitowany). Chcesz zmergować gałąź `colleague/finished-feature` do main, ale nie chcesz tracić swojej niedokończonej pracy. Jak zachować zmiany na bok, zrobić merge i je przywrócić?

### Co ma być na GitHubie po zadaniu

* [ ] Repo `git-zad10-sledztwo` publiczne.
* [ ] Gałąź `main` zawiera **wszystkie 12 commitów** (4 z części a + 8 z części b) + dodatkowo merge commit z `colleague/finished-feature` + ewentualny commit naprawiający literówkę `Pi → PI`.
* [ ] Plik `wip.txt` na main zacommitowany lub `git stash drop`-nięty (wedle wyboru -- udokumentuj decyzję).

### Pytania (do RAPORT.md)

Dla **każdego z 3 problemów** wpisz:

1. **Problem (a) -- "stracone" commity:**
    - Jakie narzędzie użyłeś?
    - Co dokładnie zaobserwowałeś (jaki output)?
    - Jak rozwiązałeś (pełna komenda CLI **lub** opis kliknięć w IntelliJ)?

2. **Problem (b) -- bug w jednym z 8 commitów:**
    - Jakie narzędzie użyłeś?
    - Ile kroków potrzebowałeś?
    - Hash złego commita.
    - Jak rozwiązałeś?

3. **Problem (c) -- niedokończone zmiany + merge gałęzi kolegi:**
    - Jakie narzędzie użyłeś?
    - Co stało się z `wip.txt` po Twoich akcjach?
    - Jak wyglądała kolejność komend?

---

## 11. Git Blame -- kto zmienił tę linię

**Cel:** Nauczyć się identyfikować autora konkretnej linii kodu i datę jej ostatniej zmiany za pomocą `git blame` oraz odpowiednika w IntelliJ IDEA (*Annotate with Git Blame*).

**Teoria w pigułce:**

* `git blame <plik>` pokazuje przy każdej linii pliku **hash commita**, **autora** i **datę** ostatniej modyfikacji tej linii.
* Blame odpowiada na pytanie *"kto i kiedy ostatnio dotknął tej konkretnej linii?"*, ale **nie** mówi nic o oryginalnym autorze logiki -- jeżeli ktoś zmienił tylko komentarz w okolicy, dla blame'a będzie "winowajcą" całej linii.
* W przypadku regresji behawioralnej *("od kiedy program się zepsuł?")* używamy `git bisect`, a `git blame` służy do śledzenia **autorstwa linii**.
* W IntelliJ: prawy klik na marginesie z numerami linii → *Annotate with Git Blame*. Kliknięcie w hash otwiera diff commita.

### Krok po kroku

```bash
# 1. Przygotowanie repo
mkdir git-zad11-blame && cd git-zad11-blame
git init
git branch -M main

# 2. Pierwszy autor (Twój e-mail) dodaje plik
git config user.name "Autor A"
git config user.email "autor.a@example.com"

cat > UserService.java << 'EOF'
public class UserService {
    public void register(String email) {
        // walidacja email
        if (email == null || email.isBlank()) {
            throw new IllegalArgumentException("email required");
        }
    }

    public void delete(String email) {
        System.out.println("usuwam: " + email);
    }
}
EOF
git add UserService.java
git commit -m "feat: początkowa wersja UserService"

# 3. Drugi autor zmienia linię walidacji
git config user.name "Autor B"
git config user.email "autor.b@example.com"

sed -i.bak 's|throw new IllegalArgumentException("email required");|throw new IllegalArgumentException("email cannot be empty");|' UserService.java && rm UserService.java.bak
git add UserService.java
git commit -m "feat: bardziej opisowy komunikat walidacji"

# 4. Trzeci autor zmienia komentarz tuż nad linią z bugiem
git config user.name "Autor C"
git config user.email "autor.c@example.com"

sed -i.bak 's|// walidacja email|// walidacja: e-mail nie może być pusty|' UserService.java && rm UserService.java.bak
git add UserService.java
git commit -m "docs: doprecyzuj komentarz walidacji"

# 5. Blame -- kto jest "winny" każdej linii
git blame UserService.java

# 6. Blame z ograniczeniem do konkretnych linii (np. 3-6)
git blame -L 3,6 UserService.java

# 7. Blame ignorujący zmiany białych znaków (formatowanie)
git blame -w UserService.java

# 8. Pokaż commit przypisany do linii 5
git blame -L 5,5 UserService.java
# Skopiuj hash i obejrzyj cały commit:
# git show <hash>
```

W IntelliJ:

1. Otwórz `UserService.java`.
2. Prawy klik na marginesie z numerami linii → **Annotate with Git Blame**.
3. Najedź kursorem na hash przy linii -- zobaczysz datę, autora i wiadomość commita.
4. Kliknij hash, aby otworzyć diff tego commita.

### Pytania kontrolne

1. Która komenda pokaże autorów tylko linii 4-7 pliku `UserService.java`?
2. Załóż, że ostatni commit zmienił tylko biały znak (tab → spacja) w danej linii. Czy `git blame` przypisze tę linię do tego commita? A `git blame -w`?
3. Czym różni się odpowiedź `git blame` od odpowiedzi `git bisect` na pytanie *"kto wprowadził buga"*?
4. W IntelliJ -- jak włączyć widok *Annotate with Git Blame* i jak go wyłączyć?
5. Czy `git blame` modyfikuje historię lub repo? Jakie ma efekty uboczne?

---

## 12. Git Diff zaawansowane

**Cel:** Opanować różne tryby `git diff` -- porównanie gałęzi, ograniczenie do pliku, statystyki zmian oraz diff tego, co jest w stagingu.

**Teoria w pigułce:**

* `git diff` bez argumentów pokazuje zmiany **niestaged** (working dir vs staging).
* `git diff --staged` (alias: `--cached`) pokazuje zmiany **w staging** względem ostatniego commita -- czyli to, co realnie pójdzie w następny commit.
* `git diff A..B` pokazuje różnice między dwoma referencjami (gałęzie, commity, tagi).
* `git diff A...B` (trzy kropki) pokazuje różnice **od wspólnego przodka** do `B` -- użyteczne przy code review PR-ów.
* `git diff --stat` daje statystyki (ile linii dodano/usunięto w każdym pliku) bez treści.
* `git diff <ref> -- <plik>` ogranicza wynik do jednego pliku/katalogu.

### Krok po kroku

```bash
# 1. Repo bazowe
mkdir git-zad12-diff && cd git-zad12-diff
git init
git branch -M main

cat > app.py << 'EOF'
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b
EOF
cat > README.md << 'EOF'
# Aplikacja kalkulatorowa
EOF
git add .
git commit -m "init"

# 2. Gałąź feature/xyz -- zmiana w 2 plikach
git switch -c feature/xyz

cat > app.py << 'EOF'
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b
EOF
cat > README.md << 'EOF'
# Aplikacja kalkulatorowa

Obsługuje dodawanie, odejmowanie i mnożenie.
EOF
git add .
git commit -m "feat: mnożenie + opis"

# 3. Wprowadź zmiany NIE-commited (working dir)
echo "" >> app.py
echo "def divide(a, b):" >> app.py
echo "    return a / b" >> app.py

# 4. Część zmian wrzucamy do staging
git add app.py

# 5. Dodaj NIE-staged zmianę w README.md
echo "Roadmap: dzielenie, potęgowanie." >> README.md

# 6. git diff (zmiany NIE-staged)
git diff
# Pokazuje tylko README.md (bo app.py jest już w staging)

# 7. git diff --staged (co pójdzie w następny commit)
git diff --staged
# Pokazuje app.py (dodane "def divide")

# 8. git diff A..B (różnice między gałęziami)
git diff main..feature/xyz

# 9. Ograniczenie do jednego pliku
git diff main..feature/xyz -- app.py

# 10. Statystyki zmian (bez treści diff)
git diff --stat main..feature/xyz
# Wynik np.:
#  README.md | 2 ++
#  app.py    | 3 +++
#  2 files changed, 5 insertions(+)

# 11. Diff od wspólnego przodka (trzy kropki) -- różnica vs dwie kropki
git diff main...feature/xyz

# 12. Diff samego commita (zmiana wprowadzona przez ten commit)
git diff HEAD~1..HEAD

# 13. Diff z kolorem słów (zamiast linii) -- ułatwia czytanie krótkich zmian
git diff --word-diff main..feature/xyz
```

W IntelliJ:

1. *Git → Show Git Log* -- prawy klik na commicie → **Compare with Local** lub **Compare with Branch…** → wybierz drugą gałąź.
2. Aby zobaczyć staged vs working dir: *Git → Commit* (`Ctrl+K`) -- IntelliJ wyświetli diff w dolnym panelu i pozwoli przełączać między widokami.

### Pytania kontrolne

1. Wykonałeś `git add file.txt`, ale nie zrobiłeś commita. Jakie polecenie pokaże Ci dokładnie zawartość, która pójdzie w następny commit?
2. Czym różni się `git diff main..feature` od `git diff main...feature` (dwie kropki vs trzy)? Kiedy używać której formy?
3. Jak wyświetlić statystyki (`+`/`-` per plik) bez pełnej zawartości diffa?
4. Jakim poleceniem porównasz tylko jeden plik (`app.py`) między dwiema gałęziami?
5. Jak wyświetlić diff ostatniego commita (zmiany wprowadzone przez `HEAD`)?

---

## 13. Force push z `--force-with-lease`

**Cel:** Bezpiecznie wypchnąć przepisaną historię (po `rebase -i`, `commit --amend`, `reset`) bez ryzyka nadpisania pracy współpracownika.

**Teoria w pigułce:**

* `git push --force` (alias: `-f`) **nadpisuje** zdalną gałąź lokalną wersją bez żadnych zabezpieczeń -- jeśli ktoś dopchnął nowy commit po Twoim ostatnim `fetch`, **stracisz jego pracę**.
* `git push --force-with-lease` najpierw sprawdza, czy zdalna gałąź jest dokładnie w stanie, jaki Git zapamiętał przy ostatnim `fetch`. Jeśli ktoś coś dopchnął w międzyczasie -- push **zostaje odrzucony**, a Ty dostajesz szansę pobrać i obejrzeć tę zmianę.
* W IntelliJ *Force Push* domyślnie używa `--force-with-lease`.
* **NIGDY** nie używaj force push na gałęziach współdzielonych (`main`, `master`, `develop`) bez wcześniejszego ustalenia z zespołem.

### Krok po kroku

```bash
# 1. Repo bazowe
mkdir git-zad13-force-push && cd git-zad13-force-push
git init
git branch -M main

echo "v1" > file.txt
git add file.txt
git commit -m "Commit A"
echo "v2" > file.txt
git add file.txt
git commit -m "Commit B"

# Załóż remote (jeśli nie masz, utwórz na GitHubie git-zad13-force-push)
# git remote add origin https://github.com/<TWOJ-USER>/git-zad13-force-push.git
git push -u origin main

# 2. Symuluj "amend" -- zmień ostatni commit
echo "v2 poprawione" > file.txt
git add file.txt
git commit --amend -m "Commit B (poprawiony)"

# 3. Próba zwykłego pusha -- ZOSTANIE ODRZUCONA
git push
# rejected -- non-fast-forward

# 4. BEZPIECZNY force push
git push --force-with-lease
# powinno przejść, bo nikt nie dopchnął nic w międzyczasie

# 5. Symulacja: ktoś dopchnął nowy commit z innego klienta.
#    Tworzymy drugi katalog roboczy, klonujemy, dopychamy commit.
cd ..
git clone <URL-zdalnego-repo> kolega
cd kolega
echo "Linia kolegi" >> file.txt
git add file.txt
git commit -m "feat: dopchnięcie kolegi"
git push

# 6. Wróć do swojego katalogu i spróbuj force-with-lease (BEZ fetch)
cd ../git-zad13-force-push
echo "moja kolejna zmiana" > file.txt
git add file.txt
git commit --amend -m "Commit B (jeszcze raz)"

git push --force-with-lease
# rejected -- stale info (Git wykrył, że origin ma nowszy stan niż pamiętałeś)
# Komunikat ratuje przed nadpisaniem pracy kolegi.

# 7. Porównaj z brutalnym force pushem (NIE rób tego w prawdziwym projekcie!)
# git push --force          # to NADPISAŁOBY commit kolegi -- nigdy tak nie rób

# 8. Poprawne działanie: pobierz zmiany, zintegruj, push.
git fetch origin
git rebase origin/main           # zintegruj zmianę kolegi
git push --force-with-lease      # tym razem przejdzie

# 9. Bonus -- jawne sprecyzowanie oczekiwanego stanu
# Sprawdź lokalnie, jaki hash uważasz za "ostatnio widziany" na origin:
git rev-parse origin/main
# Następnie wymuś przewidywaną wersję:
git push --force-with-lease=main:<expected-hash>
```

W IntelliJ:

1. *Git → Push…* (`Ctrl+Shift+K`).
2. Jeżeli zwykły push się nie powiedzie, kliknij strzałkę obok przycisku **Push** → **Force Push**.
3. IntelliJ pod spodem wywoła `git push --force-with-lease`.

### Pytania kontrolne

1. Czym dokładnie różni się `--force` od `--force-with-lease`? Który chroni przed nadpisaniem cudzej pracy?
2. Po jakich operacjach (wymień min. 3) jesteś **zmuszony** użyć force pusha, a kiedy wystarczy zwykły `push`?
3. Czy `--force-with-lease` zadziała, jeżeli przed pushem zrobiłeś `git fetch origin`, ale nie zauważyłeś nowych commitów? Dlaczego?
4. Jak w IntelliJ uruchomić *Force Push*? Czy używa `--force` czy `--force-with-lease`?
5. Wymień 2 gałęzie, na których w pracy zespołowej **nigdy** nie powinno się robić force pusha (i powiedz, jak je chronić w UI GitHuba).

---

## 14. Git Alias -- skróty dla codziennych komend

**Cel:** Skonfigurować własne aliasy, aby często używane komendy Git wykonywać szybciej i z mniejszą liczbą literówek.

**Teoria w pigułce:**

* `git config --global alias.<skrót> "<komenda>"` zapisuje alias do `~/.gitconfig` -- działa we wszystkich repozytoriach.
* `git config --local alias.<skrót> "..."` zapisuje alias tylko w bieżącym repo (`.git/config`).
* Alias może wskazywać na komendę Gita (`alias.st = status`) lub na **dowolne polecenie shell** poprzedzone `!` (`alias.lg = !git log --oneline --graph`).
* Aliasy są tylko **lokalne** -- nie są wysyłane na remote i nie są dostępne dla zespołu, chyba że masz wspólny plik konfiguracyjny.
* W IntelliJ aliasy CLI nie są widoczne w UI, ale działają w Terminalu IntelliJ (`Alt+F12`).

### Krok po kroku

```bash
# 1. Sprawdź obecne aliasy
git config --global --get-regexp '^alias\.'

# 2. Podstawowe aliasy (skróć najczęstsze komendy)
git config --global alias.st "status"
git config --global alias.co "checkout"
git config --global alias.sw "switch"
git config --global alias.br "branch"
git config --global alias.ci "commit"

# 3. Bardziej rozbudowane aliasy
# Ładny log z grafem
git config --global alias.lg "log --oneline --graph --all --decorate"

# Log z autorami i datami
git config --global alias.hist "log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short"

# Zaktualizowany "status" -- krótki format
git config --global alias.s "status -s"

# Ostatni commit -- pełne info
git config --global alias.last "log -1 HEAD --stat"

# Reset --soft (cofnij ostatni commit, zostaw zmiany w staging)
git config --global alias.undo "reset --soft HEAD~1"

# Diff w staging
git config --global alias.staged "diff --staged"

# Lista wszystkich aliasów
git config --global alias.aliases "config --get-regexp '^alias\\.'"

# 4. Alias wywołujący komendę shell (z prefiksem !)
# Pokaż wielkość repo
git config --global alias.size "!du -sh .git"

# Aktualizuj wszystkie remoty
git config --global alias.up "!git fetch --all --prune && git pull --rebase"

# 5. Test aliasów
mkdir git-zad14-alias && cd git-zad14-alias
git init
echo "hello" > test.txt
git add test.txt
git ci -m "init"          # zamiast git commit -m
git st                     # zamiast git status
git lg                     # zamiast git log --oneline --graph --all --decorate
git last                   # ostatni commit z diff statem
git aliases                # lista wszystkich Twoich aliasów

# 6. Usuwanie aliasu
git config --global --unset alias.staged

# 7. Lokalny alias (tylko w bieżącym repo)
git config --local alias.deploy "!echo deploying..."
git deploy
# Działa TYLKO w tym repo

# 8. Sprawdź, gdzie alias jest zapisany
cat ~/.gitconfig | grep -A 20 '^\[alias\]'
```

W IntelliJ:

1. Otwórz Terminal IntelliJ (`Alt+F12`).
2. Wszystkie aliasy działają tak samo jak w Git Bash.
3. UI IntelliJ ignoruje aliasy -- używa wewnętrznych komend.

### Pytania kontrolne

1. Jaka jest różnica między `--global` a `--local` przy definiowaniu aliasu? Gdzie zapisywany jest każdy z nich?
2. Co oznacza prefiks `!` w aliasie (np. `alias.up = !git fetch ...`)? Czego on **nie pozwala** zrobić bez tego prefiksu?
3. Jak wyświetlić listę wszystkich zdefiniowanych aliasów?
4. Czy alias `git ci` zostanie automatycznie przesłany do współpracowników, gdy zrobisz `git push`? Dlaczego?
5. Zaproponuj 3 aliasy, które dla Ciebie osobiście miałyby największy sens w codziennej pracy. Uzasadnij wybór.

---

## Materiały do doczytania

* **Pro Git book (PL):** [https://git-scm.com/book/pl/v2](https://git-scm.com/book/pl/v2)
* **Atlassian Git tutorials:** [https://www.atlassian.com/git/tutorials](https://www.atlassian.com/git/tutorials)
* **GitHub Docs -- About Pull Requests:** [https://docs.github.com/en/pull-requests](https://docs.github.com/en/pull-requests)
* **IntelliJ IDEA -- Set up a Git repository:** [https://www.jetbrains.com/help/idea/set-up-a-git-repository.html](https://www.jetbrains.com/help/idea/set-up-a-git-repository.html)
* **Oh Shit, Git!?!** -- typowe wpadki i jak je naprawić: [https://ohshitgit.com/](https://ohshitgit.com/)
* **GitHub -- Authentication:** [https://docs.github.com/en/authentication](https://docs.github.com/en/authentication)

---

> **Powodzenia!** Jeśli utkniesz na jakimś zadaniu -- nie pomijaj go. Wpisz w RAPORT.md "utknąłem na kroku X, próbowałem Y, dostałem Z" -- to też jest cenne. Lepiej spróbować i pomylić niż pominąć.
