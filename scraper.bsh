#!/bin/bash
#set -xv
#Gaël A et Daniel L
# Récupére les dernière sortie de film depuis différentes sources et en fait
# une page html. 
# TP UNIX de première année à l'ENSISA
# PARAMETRES ################################################################
ALLOCINE_URL="http://www.allocine.fr/film/sorties-semaine/"
PREMIERE_URL="http://www.premiere.fr/Cinema/Films-et-seances/Sorties-Cinema"
SENSCRITIQUE_URL="http://www.senscritique.com/films/en-salle/cette-semaine"
DOWN_DIR=downloaded
PARSE_DIR=parsed
PARSE_DIR_ALLO=$PARSE_DIR/allo
PARSE_DIR_PREM=$PARSE_DIR/prem
PARSE_DIR_SENS=$PARSE_DIR/sens
#############################################################################

init() {
	[ ! -d $DOWN_DIR ] && mkdir $DOWN_DIR 
	[ ! -d $PARSE_DIR ] && mkdir $PARSE_DIR
	[ ! -d $PARSE_DIR_ALLO ] && mkdir $PARSE_DIR_ALLO
	[ ! -d $PARSE_DIR_PREM ] && mkdir $PARSE_DIR_PREM
	[ ! -d $PARSE_DIR_SENS ] && mkdir $PARSE_DIR_SENS
}

nettoyeur() {
	rm -r $PARSE_DIR_ALLO/* $PARSE_DIR_PREM/* $PARSE_DIR_SENS/* $DOWN_DIR/* 
}

dl() {
	dl1 ; dl2 ; dl3 ;
}

dl1() {
	curl -# $ALLOCINE_URL > $DOWN_DIR/allocine.html
}

dl2() {
	curl -# $PREMIERE_URL > $DOWN_DIR/premiere.html 
}

dl3() {
	curl -# $SENSCRITIQUE_URL > $DOWN_DIR/senscritique.html 
}

analyse_allo() {
	for i in `grep -n '"data_box"' $DOWN_DIR/allocine.html | cut -d: -f 1`; do
	    val=`expr "$i" + 110`
	    sed -n "$i,${val}p" $DOWN_DIR/allocine.html > $PARSE_DIR_ALLO/$i
	    sed -n '/"datePublished"/p' $PARSE_DIR_ALLO/$i | cut -d '>' -f 2 | cut -d '<' -f 1 > $PARSE_DIR_ALLO/{$i}_date
	    sed -n -e "/"titlebar*"/,/"titlebar*"/p" $PARSE_DIR_ALLO/$i > $PARSE_DIR_ALLO/tmp_titre
	    sed "/^</ d" $PARSE_DIR_ALLO/tmp_titre | head -n 1 > $PARSE_DIR_ALLO/{$i}_titre ; rm $PARSE_DIR_ALLO/tmp_titre ;
	    grep -m 1 "src='http" $PARSE_DIR_ALLO/$i | cut -d "'" -f 2 > $PARSE_DIR_ALLO/{$i}_img
        sed -n '/<span/p' $PARSE_DIR_ALLO/$i | grep Réalisateur -A 1 | cut -d ">" -f 2 | cut -d "<" -f 1 | tail -n 1 > $PARSE_DIR_ALLO/{$i}_realisateur
	    sed -n '/<span/p' $PARSE_DIR_ALLO/$i | grep genre |  cut -d ">" -f 2 | cut -d "<" -f 1 > $PARSE_DIR_ALLO/{$i}_genre
	    grep note $PARSE_DIR_ALLO/$i | sed -n "s/.*>\(.*\)<\/span><\/span><\/div>/\1/p" > $PARSE_DIR_ALLO/{$i}_note
        head -n 1 $PARSE_DIR_ALLO/{$i}_note > $PARSE_DIR_ALLO/{$i}_note_press
        tail -n 1 $PARSE_DIR_ALLO/{$i}_note > $PARSE_DIR_ALLO/{$i}_note_spect ; rm $PARSE_DIR_ALLO/{$i}_note;	
    done
}

analyse_prem() {
	for i in `grep -n '<div class="texte">' $DOWN_DIR/premiere.html | cut -d: -f 1`;do
	    val=`expr "$i" + 13`
	    sed -n "$i,${val}p" $DOWN_DIR/premiere.html > $PARSE_DIR_PREM/$i
        sed $PARSE_DIR_PREM/$i -n -e "s/.*>\(.*\)<\/a><\/div><div.*/\1/p" | head -n 1 >$PARSE_DIR_PREM/{$i}_titre
        grep "infos_notes" $PARSE_DIR_PREM/$i | sed -n  "s/\(.*\)<\/li.*/\1/p" | cut -d ":" -f 2 > $PARSE_DIR_PREM/{$i}_pays
        grep "infos" $PARSE_DIR_PREM/$i | sed -n  "s/.*<ul><li>\(.*\)<em.*/\1/p" > $PARSE_DIR_PREM/{$i}_genre
	    grep "picto_etoile " $PARSE_DIR_PREM/$i | sed -n "s/.*>\(.*\)<\/em><em.*/\1/p" > $PARSE_DIR_PREM/{$i}_note_spect 
    done
}

analyse_sens() {
	for i in `grep -n '"elpr-item"' $DOWN_DIR/senscritique.html | cut -d: -f 1`; do
	    val=`expr "$i" + 110`
	    sed -n "$i,${val}p" $DOWN_DIR/senscritique.html > $PARSE_DIR_SENS/$i
	    sed -n "/"elco-title"/,/"span"/p" $PARSE_DIR_SENS/$i > $PARSE_DIR_SENS/tmp_titre
	    sed -n "s/.*>\(.*\)<.*/\1/p" $PARSE_DIR_SENS/tmp_titre | head -n 1 > $PARSE_DIR_SENS/{$i}_titre ; rm $PARSE_DIR_SENS/tmp_titre;
	    sed $PARSE_DIR_SENS/$i -n -e "s/.*>\(.*\)<\/time>/\1/p" | cut -d "." -f 1 > $PARSE_DIR_SENS/{$i}_date
	    sed $PARSE_DIR_SENS/$i -n -e "s/.*\( .* h .*\).*/\1/p" | cut -d "." -f 1 > $PARSE_DIR_SENS/{$i}_duree
	    grep -A 3 erra-rating $PARSE_DIR_SENS/$i | sed -n -e "s/.*\( .*\)<\/a>/\1/p" | cut -d ">" -f 2 > $PARSE_DIR_SENS/{$i}_note
    done
}

analyse() {
	[ -s "$DOWN_DIR/allocine.html" ] && analyse_allo 
	[ -s "$DOWN_DIR/premiere.html" ] && analyse_prem 
	[ -s "$DOWN_DIR/senscritique.html" ] && analyse_sens
}

allo_html() {
    [ ! -d $PARSE_DIR_ALLO/final ] && mkdir $PARSE_DIR_ALLO/final
    for i in `ls $PARSE_DIR_ALLO/[!{f]* | sort -g | cut -d\/ -f 3`; do 
		{
        echo "<article><h2>"
        echo "`cat $PARSE_DIR_ALLO/{$i}_titre`"; echo "</h2>"
        echo "<img src=\"`cat $PARSE_DIR_ALLO/{$i}_img`\"/>"
        echo "<ul>" 
        echo "<li>Date de sortie : `cat $PARSE_DIR_ALLO/{$i}_date`"; echo "</li>"
        echo "<li>Note presse : `cat $PARSE_DIR_ALLO/{$i}_note_press`"; echo "</li>"
        echo "<li>Note spectateur : `cat $PARSE_DIR_ALLO/{$i}_note_spect`"; echo "</li>"
        echo "<li>Réalisateur : `cat $PARSE_DIR_ALLO/{$i}_realisateur`"; echo "</li>"
        echo "<li>Genre(s) : `cat $PARSE_DIR_ALLO/{$i}_genre`"; echo "</li>"
        echo '</ul>'
        echo "</article>"
		}> $PARSE_DIR_ALLO/final/"`cat $PARSE_DIR_ALLO/{$i}_titre`";
    done
}

prem_html() {
    [ ! -d $PARSE_DIR_PREM/final ] && mkdir $PARSE_DIR_PREM/final
    for i in `ls $PARSE_DIR_PREM/[!{f]* | sort -g | cut -d\/ -f 3`; do 
        {
        echo "<article><h2>" 
        echo "`cat $PARSE_DIR_PREM/{$i}_titre`"  ; echo "</h2>" ;
        echo "<ul>" ;
        echo "<li>Pays de production : `cat $PARSE_DIR_PREM/{$i}_pays`" ; echo "</li>"
        echo "<li>Note spectateur : `cat $PARSE_DIR_PREM/{$i}_note_spect`"; echo "</li>"
        echo "<li>Genre(s) : `cat $PARSE_DIR_PREM/{$i}_genre`"; echo "</li>"
        echo '</ul>'
        echo "</article>"
		}> $PARSE_DIR_PREM/final/"`cat $PARSE_DIR_PREM/{$i}_titre`";
    done
}

sens_html() {
    [ ! -d $PARSE_DIR_SENS/final ] && mkdir $PARSE_DIR_SENS/final
    for i in `ls $PARSE_DIR_SENS/[!{ft]* | sort -g | cut -d\/ -f 3`; do 
        {
        echo "<article><h2>" 
        echo "`cat $PARSE_DIR_SENS/{$i}_titre`"; echo "</h2>" 
        echo "<ul>" 
        echo "<li>Note : `cat $PARSE_DIR_SENS/{$i}_note`"; echo "</li>"
        echo "<li>Date de Sortie : `cat $PARSE_DIR_SENS/{$i}_date`"; echo "</li>"
        echo "<li>Durée : `cat $PARSE_DIR_SENS/{$i}_duree`"; echo "</li>"
        echo '</ul>'
        echo "</article>"
        }> $PARSE_DIR_SENS/final/"`cat $PARSE_DIR_SENS/{$i}_titre`";
    done
}

html() {
    [ -s "$DOWN_DIR/allocine.html" ] && allo_html
	[ -s "$DOWN_DIR/premiere.html" ] && prem_html
	[ -s "$DOWN_DIR/senscritique.html" ] && sens_html
}

base_html() {
	{
    echo "<!DOCTYPE html>" 
    echo '<html lang="fr">'
    echo '<head><meta http-equiv="content-type" content="text/html; charset=UTF-8"><meta charset="utf-8"><title>Scraper Alberola & Lazzaro</title>'
    echo '<style>*{text-align: center; background-color:light-grey;}</style>'
	}> $nom_page_html
}

contenu() {
	cat $PARSE_DIR_ALLO/final/* >> $nom_page_html
	echo '<h1>Donnée de Première</h1>' >> $nom_page_html
	cat $PARSE_DIR_PREM/final/* >> $nom_page_html
	echo '<h1>Donnée de SensCritique</h1>' >> $nom_page_html
	cat $PARSE_DIR_SENS/final/* >> $nom_page_html
}

fin_html() {
    echo "</body></html>" >> $nom_page_html
}

while [ $# -ne "0" ]; do
	case "$1" in
		"-i")
			init;
			;;
		"-c")
			nettoyeur;
			;;
		"-s")
			if [[ $2 == -* || -z $2 ]]; #test pour les cas "-s -i" ou "-s"
			then
				dl;
			fi
			while [[ $2 != -* ]]; do  # test pour les cas " -s premiere 1 2 3 -i"
				if [ -z $2 ]; #alléluhia -> on quite la boucle si on a "-s premiere 1 2" (non suivi de -*)
				then
					break
				fi
				case "$2" in 
				"allocine" | 1)
					dl1;
					;;
				"premiere" | 2)
					dl2;
					;;
				"senscritique" | 3)
					dl3;
					;;
				esac
				shift
			done
			;;
		"-a")
			analyse;
			;;
		"-h")
			nom_page_html=$2
			allo_html;prem_html;sens_html;
			base_html;contenu;fin_html;
			;;
		"-w")
			nom_page_html=$2
			allo_html;prem_html;sens_html;
			base_html;contenu;fin_html;
			;;
	esac
	shift
done
