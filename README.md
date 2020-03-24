dinosaure-computing
===================

** compilation of snippets in computing with prehistoric OS. howto script and compile recent opensources **


[original : http://guykastenbaum.blogspot.ch/2012/01/scripter-comme-un-dinosaure.html 27 janvier 2012]

comme j’aime bien cet article, je l’ai retrouvé sur blogger et réinjecté dans mon blog actuel.

Comment scripter (sh ou ksh) sur des environnements compatibles linux AIX Solaris ?

Je sais qu’il suffirait de changer de versions de sh ksh bash etc mais

## if 

Quelques recettes de cuisine :pour le if

```
if [ “x${var}” = “xyes” ]; then echo “yes”; fi
```

RESPECTER les espaces, non il ne faut rien coller, ni [“, ni “=”, ah c’est dur…

ajouter le “x” pour éviter les if “-u” …=>; erreur sur l’opération -u

ajouter les guillemets car dans if [ $var =  $var pourrait être vide
(ça fait doublon avec “x” mais on est jamais assez prudent)

de temps en temps ajouter les accolades pour $var , soit ${var}, par pure paranoia

ajout récent : dans les tests d’égalité, il faut utiliser ‘=’ et non ‘ = = ‘(==), c’est pas compatible avec le vieux ‘sh’;

utiliser -eq plutôt que == , quoique on peut quand même oser. (en général, j’ose, c’est plus lisible)

je n’utilise pas le raccourci $a == “y” || $b=”n” , j’ai trop peur que ça soit trop moderne.

avec prudence les OR et AND sous forme de -o et -a ;

les jours de grande parano, utiliser des doubles crochets comme dans la doc

```
if [[ “x$a” != “xy” -a “x$a” != “xn” ]];then echo “maybe”; fi
```

pour faire un $x=~/toto/ [perl] ben d’à peu près lisible j’ai

```
if [ ” `echo $x|sed -e 's/toto//'`" != "$x" ];then ...
```

## pour le sed 

en général il est sympa, sauf chez solaris

sur solaris sed -e ‘s/\t/ /g’ remplace les ‘t’ par des blancs.(oui,oui)

il faut remplacer sed -e “s/\t/ /g” par tr ‘\t’ ‘ ‘

c’est ce qui m’a décidé à écrire ce billet
d’ailleurs, tr marche assez bien : tr [:upper:] [:lower:] est bien compris

cadeau pour générer un mot de passe de 6 caractères :

```
$(head -n 100 /dev/urandom| tr -dc “[A-Z][a-z][0-9]”|cut -c1-6)
```

de même, si tu as des fichiers qui ne se termine pas par “\n” , le sed te zappe la dernière ligne. (oui,oui)

faire une boucle de réparation avant :

```
for instfile in $(find . -type f);do
if [ “$(tail -1 $instfile|od -t x1|tail -2|head -n 1|grep -c ‘ 0a’)” == “0” ];then
echo “” >> $instfile
fi
done
```

ah oui, oubliez le pratique -i, il faut revenir à sed -e ‘….’ a>b;mv b a

ou mieux car il conserve les attributs cp -p a b;sed -e ‘..’ b>a;rm -f b

## [ajout de mai 2012] sed

Encore Solaris qui nous rappelle que sed -e “s/a/b/ s/c/d/” est un raccourci trop récent pour être supporté. Pour être 19k-compatible il faut écrire : sed -e “s/a/b/” -e “s/c/d/”

## [ajout avril 2013]

Ah AIX se fait remarquer, on ne peut pas faire sed fichier -e ‘…’

car ça n’est pas dans le bon ordre. Soit sed -e ‘…’ fichier soit (c’est plus sûr)

```
cat fichier|sed -e ‘…’
```

## ed

quelquefois on s’en sort en allant chez cet épicier discount, exemple

```
ed -s src/xmlrpc_introspection.h << EOD
/XMLRPC_ServerSetValidationLevel
i
XMLRPC_VALUE find_named_value(XMLRPC_VALUE list, const char* needle);
void describe_method(XMLRPC_SERVER server, XMLRPC_VALUE vector, const char* method);
.
w
q
EOD
```
 
 
## shell (sh echo $* .)

ne pas utiliser le trop moderne ‘-n’ option d’echo, car

```
echo “echo -n toto” > /tmp/a ; sh /tmp/a ; ksh /tmp/a ; rm a
```

affiche bien “-n toto”

éviter les echo avec des passages à la ligne , même avec \

sh ne connait pas $(commande), il ne connait que `commande` , c’est pas pratique.

sh n’accepte pas export PIF=”paf” , il exige PIF=”paf” ; export PIF
de toutes façon j’utilise ksh (c’est les ordres de là haut)

## $0 $1 $* $@

variables fragiles, elles peuvent s’effacer le long du script (suffit qu’elles soient utilisées en sous entendu dans un sous appel), donc les sauver dès le début du script

$0 des fois contient “-bash” au lieu de “bash” je sais pas d’où ça sort.

##source 

“source file” : utiliser “. ./file” ; c’est moins lisible mais plus archaique

cadeau, un complément de getopts

```
while getopts “n:t:m:a:u:s:h” opt; do
if [ “$OPTARG” = “” ]; then OPTARG=1; fi
if [ “$opt” != ‘?’ ] && [ “$opt” != ‘:’ ]; then
export OPTARG_${opt}=$OPTARG
fi
done
```

les fonctions s’écrivent

```
compilesys()
{
```

pas de ‘function’

des parenthèses vides (on chopera $1 $2 …)

accolade à la ligne

et ça s’appelle sans les parenthèses, compilesys unzip

## pour id, uname

pour les fonctions borderline, qu’on croit qu’elles sont standards, eh ben moitié moitié.

faut vérifier les options de base. sur solaris j’utilise

```
if [ -f “/usr/xpg4/bin/id” ];then export GSCRIPT_BIN_ID=/usr/xpg4/bin/id;
else export GSCRIPT_BIN_ID=id; fi
```

sinon je ne m’en sors pas (besoin du id -gn -u -g ..)

uname -s est plus fiable que hostname (JYP)

## grep

ne connait pas toujours -E (solaris) , par contre son équivalent egrep existe …

ah, ni -r (récursif) , évidemment (mais ggrep si)

## tar

tar, oubliez le -z pour de copain gzip, retrouvez tar cf -|gzip -c|gzip -dc|tar xf –

pour les gros tar, ou avec des longs paths, utiliser gtar ou une version fraiche :

pour tar gzip zip bzip make … : RECOMPILEZ ! cadeau :

```
compilesys()
{
confplus=$2
gzip -dc ../zip/$i-*.gz 2> /dev/null | tar xf – #ou variante…
cd $i-*/ ;   ./configure –cache-file=config.cache –prefix=SOMEWHERE $confplus
make ;  make install ;  cd ..
}
compilesys tar “FORCE_UNSAFE_CONFIGURE=1”
for soft in xz bison flex m4 autoconf automake libtool make bash binutils;do compilesys $soft;done
```

## (2020) read

oh oh une nouvelle m.. subtilité

read de bash tourne dans un subprocess, au contraire de ksh, 
subtil hein, c'est grave ? ben... 

```
echo "a b c"|read A B C && echo "A=$A B=$B C=$C"

cat file|while read LINE;do A=$($A + 1) done;echo $A
```

ben en ksh ça fait ce qu'on attend. en bash non

pour retrouver la compatibilité, abracadabra

```
shopt -s lastpipe
```

