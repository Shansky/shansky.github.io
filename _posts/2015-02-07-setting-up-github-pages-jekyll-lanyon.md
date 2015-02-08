---
layout: post
title: Własna stronka na github.com
---

W tej kolejności:

1. Tutaj opisane jest jak stworzyć repozytorium na githubie, które można następnie wykorzystać jako github-page - [https://pages.github.com/](https://pages.github.com/) - w moim przypadku:
<p>*shansky.github.io*</p>
2. Clone repozytorium:

	```bash
	$ git clone git clone https://github.com/shansky/shansky.github.io
	```
3. Ustawienie zdalnego brancha [poole](https://github.com/poole/poole) z motywem [lanyon](https://github.com/poole/lanyon):

	```bash
	/shansky.github.io/ $ git remote add lanyon https://github.com/poole/lanyon.git
	```
4. Pobranie repozytorium:

	```bash
	/shansky.github.io/ $ git fetch lanyon
	```
Dla wygody jeszcze zmiana nazw:

	```bash
	/shansky.github.io/ $ git branch lanyon-master lanyon/master
	```
Pozostaje tylko merge:

	```bash
	/shansky.github.io/ $ git merge lanyon-master
	```
5. Później standardowa praca z gitem: ```bash git add```, ```bash git commit```, ```bash git push -u  origin master``` - i stronka dostępna pod adresem [https://shansky.github.io/](https://shansky.github.io/)
6. Hmm tutaj powinienem opisać strukturę projektu ... powinienem :)
7. Projekt poole bazuje na [jekyll](http://jekyllrb.com) i to z nim można pracować lokalnie, żeby sprawdzić jak strona zostanie sparsowana/zbudowana. Jekyll można zainstalować z 'dżemów':
	```bash
	$ gem install jeckyll
	```
Jeśli przy odpaleniu ```bash jeckyll --help``` pojawi się wężyk błędów, zaczynający się od lini zawierającej *Could not find a JavaScript runtime.* - wiedz że coś się dzieje, a mianowicie potrzebujesz jakiegoś JS runtime. Najszybciej:
	```bash
	$ apt-get install nodejs
	```
I powinno Jekyllowi grać. Później będąc w katalogu z projektem (repozytorium) budujesz projekt przy użyciu ```bash jeckyll serve``` i na [http://localhost:4000/](http://localhost:4000) dostajesz zbudowaną stronkę. Gdy zapiszesz zmianę w plikach projektu, jeckyll sam odświeży zawartość projektu.
8. Dodanie własnego aliasu dla domeny *shansky.github.io*

	***to be continued...***



I właśnie otrzymałeś fajną platformę mikroblogową.

### Source:
1. [Github Pages](https://pages.github.com/)
2. [Poole project](https://github.com/poole/poole)
3. [How I Created a Beautiful and Minimal Blog Using Jekyll, Github Pages, and poole](http://joshualande.com/jekyll-github-pages-poole/)
4. [Setting the DNS for GitHub Pages on Namecheap](http://davidensinger.com/2013/03/setting-the-dns-for-github-pages-on-namecheap/)
