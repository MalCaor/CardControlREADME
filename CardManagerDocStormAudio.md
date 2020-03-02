# Card Manager - Doc :
*storm Audio*
*v0.1*

---

## Comment ça Marche?

le code est divisé en deux : **card-Control.php** et **card-Change.php**

>**card-Control** :
> Page vue par l'utilisateur
> elle fonctionne en appelant des fonction dans le fichier **vue**
> *attention ce n'est pas du mvc mais le code est tout de même découpé en fonctionalité*
>> **SNapp.php** :
>> affiche et modifi les inforamtions de Main Board,
>> *Main Board Information :*
>> Current case information : [info]
>> Current mainboard information : [info]
>> [Modif]
> ---
>> **Etat.php** :
>> affiche les info de chaque carte ainsi que la posibilité de les Exporter/Importer via csv
>> *Card :*
>> [Export]
>> [Import] [Browse...]*nom fichier*
>> [ ] Checkbox désactivant les vérifications
>> ( ) *nom carte* [info;info;info] [Apply Change]
>> ... pour chaque carte

Une fois que l'utilisateur à fini, les information sont vérifié puis sont envoyé à **card-Change.php**

>**card-Change.php** :
> cette page ne sera jamais vu par l'utilisateur, elle ne fait qu'effectuer les modifications
> - si *CardNum*, *id* et *name* sont envoyer par POST alors c'est que l'utilisateur n'a modifié qu'une seul carte, executer la requete de modification dans le daemon et dans la BDD
> - si *file* existe par POST alors l'utilisateur a importé un csv, effectue les requetes après quelles vérifications (normalement toutes les infos du csv doivent être bonne après être passé dans le js mais on n'est jamais sure)
> normalement un seul des deux cas est possible mais même si les deux était présent il ne devrai pas y a voir d'érreur, les info du csv vont juste overwrite les premières probablement
> un header vers control pour que personne ne soit bloqué sur cette page (même en rentrant sont chemin dans l'url)

>**Board-Change.php** : 
> cette page change les infos de MainBoard, pourquoi ne pas la mêtre dans **card-Change**? est-ce que main board est une carte? - voila la réponse
> si caseInfo et mainInfo exist dans POST, execute la commande pour les changer
> header vers card-Control comme pour card-Change

---

## Verification JS

**Card.js** sert principalement à la vérification des donnée et à la requete XMLHttp (et à l'affichage de la popup)
- subOK : seul var en dehors de toute fonction, la raison est qu'elle doit rester global d'une fonction à l'autre sans avoir besoin de la reset, sont but est de donner le feu vert pour le submit du csv, sub*mit*OK
- Export() : copie paste d'un truc basique qui génère un lien de téléchargement du csv puis le remove
- Import() : fonction appelé lors de l'import d'un csv
	- prend le fichier (si pas la annule tout)
	- si le fichier n'a pas d'extantion ".csv" annule tout
	- crée un nouveau fileReader et lit le fichier
	- découpe le fichier en tableau (table) puis le met en innerHTML dans le popup
	- affiche les erreurs / warning
- HideModal() :cache le popup en enlevant la classe "show-modal" à celui ci
- Submit() : si subOK est vrai, lance une requete XMLHttp à card-Change avec le fichier csv, si subOk est faux alert que le fichier est incorecte

---

## Function PHP

**Etat**
*require* : 
- js/card.js | javascript effectuant les vérifications / manage les Import Export de csv
- vue/all-vue/download.csv | fichier csv qui est remplit afin d'être potentielement téléchargé
*used* :
- card-Control.php | est appeler afin d'afficher/modifier les cartes
*input* :
- $libel,(string) | id de la div global
- $styleDiv,(string) | style de la div global
- $policeTitle,(string) | font du titre
- $policeLabelCard,(string) | font du libele de chaque carte
- $classNone,(string) | classe des carte "None" (non présente)
- $classEmpty,(string) | classe des cartes "Empty" (là mais pas écrite)
- $classHere,(string) | classe des cartes "Here" (là et écrite)
- $classButton,(string) | classe des boutons submit des cartes (Apply Change)
- $classSN,(string) | classe de l'input des info de chaque carte
- $classSVG,(string) | classe du petit rond vert ou rouge en fonction de l'état de la carte
- $circleSize,(int) | taille de ce rond (circonference)
- $ispEquivalNum,(array) | array de numéro pour chaque carte (pour que le daemon le modifi)
- $ispEquivalName,(array) | array de nom de carte pour chaque carte (car les nom diffère de la BDD au Daemon)
- $partnum_map,(array) | array de(s) partnum(s) attendu pour chaque carte 
- $array,(array) | tout les cartes fournie par la BDD (nom pas très claire)
*output* :
- $retour,(string) | long string de tout le html à afficher
*work* :
- div id=libel + "Card :"
	- importe js card.js
	- set le path du csv à écrire 
	- verification :
		- si fichier n'existe pas : die (n'existe pas)
		- si fichier impossible à écrire : die (pas les droits)
	- écrit dans le csv les info des cartes
	- bouton export avec onClick(Export())
	- form import vers /card-Change.php avec : 
		- bouton Import onClick(Import())
		- input file
		- arrayName (ne sert plus car card-Change à déja acces à cette var)
	- end form
	- div Modal (non visible sauf lors d'un import)
		- (remplit en js) table d'info de carte
		- (remplit en js) des input text de message error et warning
		- button "Cancel" onClick(HideModal()) - cache la modal
		- button "Submit" onclick(Submit()) - lance le script submit
	- end Modal
	- Checkbox verif (désactive les vérification (sauf de l'import d'un csv))
	- début foreach des cartes
		- div Carte
		- set name de la carte
		- si le nom = (HDMI, ISP_AP) : continue
		- si Value = None          : start div class None
		- si Value = p/n NA rev NA : start div class Empty
		- sinon                    : start div class Here
			- Script validateForm+$name() :
				- si Checkbox verif = checked : rien faire
				- set x the info input
				- set first la première ";" de x
				- si fist > -1 (si il exist une ";" dans x) :
					- set sec la deuxième ";"
					- si sec > -1 (si il y a une deuxième ";") :
						- set ToMany la troisième ";" de x
						- si ToMany > -1 :
							- alert : "TROP DE ;"
							- STOP
						- si sec - first = 1 :
							- alert : "il n'y a pas de numero de série"
							- STOP
						- si le dernier char de x = ";" :
							- alert : "il n'y a pas de numero de carte"
							- STOP
						- si le premier char de x = ";" :
							- demande à l'utilisateur si il veux continuer même si il n'y a pas de num de version
							- si il répond non : STOP
						- set partnum 
						- si le partnum n'est pas le bon pour la carte : STOP
						- return TRUE : OK
					- alert : "il manque un ;"
					- STOP
				- alert : "il manque deux ;"
				- STOP
			- FIN script
			- Form cardForm+Nom vers(/card-Change.php) POST onSubmit(validateForm+$name())
				- input hidden name
				- svg cercle de la couleur de l'état (None, Empty, Here)
				- Nom de la carte class=PoliceLabelCard
				- si None : input text "None" readonly
				- si Empty : input text "Empty"
				- sinon : input text "Info" (Info par un exec et vidé de toute verbose)
				- input hidden id
				- button submit
			- END From
		- END div Carte
	- END foreach
- END div globale
- return la div globale avec le string $retour 

**SNapp**
*require* :
- none
*used* :
- card-Control.php | est appeler afin d'afficher/modifier les info de Main Board
*input* :
- $classDiv,(string) | class de la div
- $policeTitle,(string) | font du titre
*output* :
- $retour,(string) | div des info de la Main Board
*work* :
- set info grace à un exec, info contient tous ce que l'on a besoin
- set posColon qui est la premier :
- set caseInfo qui est tous ce qu'il y a après ":", donc les case info
- set posColon2 qui est le deuxème ":"
- set mainInfo qui est tous ce qu'il y a après ":", donc les Main info
- div class=classDiv
	- form action=/board-Change.php POST
		- p titre "Main Board Information :" class=policeTitle
		- le texte de tout ce qui à avant : (de la verbose utils WOW)
		- input de caseInfo
		- le texte de tout ce qui à avant : (de la verbose utils WOW2)
		- input de mainInfo 
		- button submit
	- END Form
- END div
- return $retour 

**loadHTML**
require* :
- file all-vue | qui contient 
*used* :
- card-Control.php | afficher les truc de base
*input* :
- $argument,(string) | nom du fichier
*output* :
- string du fichier ou de l'erreur 
*work* :
- set path à l'endroit du fichier
- si argument vide : return "WARNING NO ARGUMENT PASS"
- si argument n'indique aucun fichier : return "WARNING NO SUCH FILE AS + argument"
- sinon : return contenu du fichier sous form de string

---

## Variable 

toutes les vars sont stockées dans vue/var et ne sont appelées que dans card-Change et card-Control
> var.js : contient toutes les variables utils pour le javaScript :
>	- arrayOfName : array des cartes existantes
>	- ispEquivalName : array des noms equivalent au même nom mais écris diff
>	- partnum_map : array de partnum pour chaque carte
>	- partnum_mapnum : la même mais avec un int

> var.php : contient toutes les variables utils pour le PHP :
>	- ispEquivalNum : array de carte et leur équivalent en numero
>	- ispEquivalName : array des noms equivalent au même nom mais écris diff
>	- partnum_map : array de partnum pour chaque carte
>	- arrayOfName : array des cartes existantes

> var.json : ne sert à rien, mais si un jour il faut fusioner les vars dans un seul json le voila