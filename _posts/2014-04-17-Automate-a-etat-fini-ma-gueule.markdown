---
layout:		post
title:		"Et un automate à états finis ma gueule !"
date:		2014-04-17 01:42:21
categories:	automate lexer langage
---

## Introduction

Putain, c'est quoi un automate à états finis ? C'est un truc que tu peux sortir devant tes potes à la pose café histoire de te la péter. Mais en réalité, c'est bien plus que ça, c'est un putain d'outil que tu utilises tout les jours, ouai toi le développeur PHP qui veut _parser_ ton HTML avec des _regexps_. Maintenant, t'arrêtes un peu tes conneries et tu m'écoutes bien pour que tu saches ce que c'est qu'un automate à états finis. Donc je sais pas si tu viens de capter mais tes _regexps_ grâce au [théorème de Kleene](http://fr.wikipedia.org/wiki/Th%C3%A9or%C3%A8me_de_Kleene), bah se sont en réalité des automates comme celui là:

![Un Putain d'automate La Poste mec, et je sais que tu l'as déjà vu]({{ site.baseurl }}/img/01.jpg)

Bon maintenant, tu le vois visuellement mais il y a quoi à l'intérieur ? C'est tout con, un automate à états finis, c'est un ensemble d'état qu'on représente avec des boules (comme les boules de ton père) et elles sont liées par des transitions (des sortes de flèches). Tu commences à partir d'un état, genre l'état `1` et puis tu vas lire un caractère. Puis en fonction de ce caractère, tu vas aller dans une transition (donc en réalité, les transitions sont caractérisées par un caractère). Et en allant dans une transition, tu vas atteindre un autre état.

Après, on a plusieurs types d'états. Genre l'état de ton père (la boule) c'est un état normal mais après il y a un état initial (note qu'il ne peut y avoir qu'un seul et unique état initial dans un automate). Puis après t'a l'état _custom_, c'est l'état final. Cette fois, tu peux en avoir plusieurs. Ces états vont servir à délivrer/signaler quelque chose, on verra ce que c'est après. Donc, t'as quoi au final ? Un automate, c'est un ensemble d'état dont un seul est initial et quelques-uns sont finaux et il y a des transitions qui sont annotés d'un caractère.

![Un vrai automate]({{ site.baseurl }}/img/02.png)

Bah là, c'est un vrai automate. Tu peux le dessiner sur le tableau de ton _open-space_, ça fait classe. Mais sinon tu les vois les boules ? Bah la boule `1`, c'est un état initial parce que t'as une flèche qui vient du vide intersidéral et qui pointe sur cet état. Ensuite t'as des transitions qui sont annoté avec des lettres (genre `a` et `b`). Et puis t'a l'état `2`, lui c'est un état final parce qu'il a un double cercle. 

T'as l'air de rien comprendre mais en vrai, cet automate à états finis, il équivaut à l'expression régulière `[ab]*b`. Tu vois maintenant ? Genre `b`, ça _match_; `abb`, ça _match_; `a`, ça _match_ pas (parce que t'es pas sur l'état final); etc.

## T'es dans la matrice

Bon t'as l'air de mieux comprendre ce que c'est. Mais alors maintenant, comment ça se passe au niveau du code ? Bah t'as plusieurs manières de représenter un automate à états finis. On va commencer facile, OK ? On va représenter un automate par une matrice. L'expression régulière qu'on aura est `din2oba` (ouai ma gueule, dédi' à [booba](https://www.youtube.com/watch?v=AFNEBA8sLJ0)). On va limiter l'alphabet sur lequel on travaille pour pas se taper les 127 caractères ASCII. Donc notre alphabet sera `din2oba` (tu peux pas test).

![Dinosaure est trop beau]({{ site.baseurl }}/img/03.png)

Bon maintenant, tu rengaines ton `strcmp`, `==`^W`===` ou ton `compare`, on va faire de l'automate ! Cela consiste à créer deux tableaux à 2 dimensions. La taille de ces tableaux sera `nombre d'états * nombre de lettres de ton alphabet` donc du `7 x 7` (49 si tu sais pas compter) - maintenant tu sais pourquoi j'ai limité l'alphabet, je me voyais pas me taper 889 cases.

Le premier tableau va être ce qu'on nomme une [matrice de transition](http://fr.wikipedia.org/wiki/Matrice_de_transition), celle-ci va tout simplement dire à qu'elle état t'es lié quand tu es sur un état `N` et que tu viens de choper le caractère `E`. Comme ça, tu seras où il faut aller en tapant `t[N][E]`. T'inquiètes pas, on va voir comment faire.

Le deuxième tableau, lui, c'est la matrice des actions. En faite, c'est là où on va différencier l'état final des autres états (en général, l'état final va dire: "putain mec, j'ai trouvé un truc qui _match_"). Sauf que cette action ne va pas se faire quand on sera sur l'état mais quand on ira vers lui. Genre, on écrira que ça _match_ quand on ira vers l'état final et pas quand on y sera (payes ta différence, mec).

Je suis un putain d'_hipster_ donc je te fais le bouzin en C++ without class (ouai, c'est le deuxième langage fait par [Bjarne Stroustrup](http://en.wikipedia.org/wiki/Bjarne_Stroustrup)) ma gueule:

{% highlight c %}
int t[8][8] = {
//  d  i  n  2  o  b  a  ?
  { 1, 0, 0, 0, 0, 0, 0, 0 },
  { 0, 2, 0, 0, 0, 0, 0, 0 },
  { 0, 0, 3, 0, 0, 0, 0, 0 },
  { 0, 0, 0, 4, 0, 0, 0, 0 },
  { 0, 0, 0, 0, 5, 0, 0, 0 },
  { 0, 0, 0, 0, 0, 6, 0, 0 },
  { 0, 0, 0, 0, 0, 0, 7, 0 },
  { 0, 0, 0, 0, 0, 0, 0, 0 },
};
{% endhighlight %}

Ici, t'imagines que les caractères de notre alphabet `din2oba` sont dans les abscisses et que tes états sont dans les ordonnées - notes que j'ai rajouté une colonne, c'est pour gérer les cas où nous choperions un caractère qui n'appartient pas à l'alphabet `din2oba`. Imagine que t'es à l'état `0` (donc la première ligne) et que tu choppes la lettre `d` (donc la première colonne). Il te dit à cet endroit précis `t[0]['d'] = 1` (on va considérer que `d` vaut `0`, hein). Donc en sachant ça, tu sais que tu vas passer à l'état `1`. Et à l'état `1`, tu pourras passer à l'état `2` seulement si tu choppes le caractère `i` (deuxième colonne). Si tu choppes une autre lettre, tu retombes sur l'état `0` (bah oui, si tu regarde la deuxième ligne, toutes les autres possibilités sont des `0`).

![LOL, ça boucle]({{ site.baseurl }}/img/04.png)

Tu l'as vois la diagonale ? En même temps, tu vas pas passer de l'état `4` vers l'état `1` (genre `t[4]['d'] = 1`) ? Sinon, mon expression régulière serait `[din2]+oba` (ouai mon gars, ça bouclerait !). Bon après notre matrice de transition, on va faire la matrice des actions, non ? Bah ouai, faut bien savoir ce qu'on fait en allant d'un état à un autre et rendre spécifique l'état final. On considère qu'il y a 3 actions possibles. La première consiste à aller basiquement d'un état à un autre mais elle va aussi enregistrer le caractère qu'on vient de lire dans un _token_. La deuxième consiste à revenir à l'état initial dans le cas d'une erreur (imaginons qu'on tombe sur le caractère `f` ou qu'on tombe d'abord sur un `i` avant de tomber sur un `d`) et supprimer le _token_ puisqu'il est faux. Enfin la dernière action consiste à caractériser l'état final. Celle ci consiste à dire que ça _match_, on délivrera notre _token_ et on pourra ensuite revenir à l'état initial.

{% highlight c %}
#define MA (0) // comme MOVE APPEND
#define ER (1) // comme ERROR
#define HR (2) // comme HALF RESET

int a[8][8] = {
  { MA, ER, ER, ER, ER, ER, ER, ER },
  { ER, MA, ER, ER, ER, ER, ER, ER },
  { ER, ER, MA, ER, ER, ER, ER, ER },
  { ER, ER, ER, MA, ER, ER, ER, ER },
  { ER, ER, ER, ER, MA, ER, ER, ER },
  { ER, ER, ER, ER, ER, MA, ER, ER },
  { ER, ER, ER, ER, ER, ER, MA, ER },
  { HR, HR, HR, HR, HR, HR, HR, HR }
};
{% endhighlight %}

## Le solveur dans ta face

Bon on a réussi à représenter notre automate à états finis en C++ avec des tableaux. Mais là, comme ça, ça sert un peu à rien. Faut un truc qui les manipule, tu vois. Avant que je balance le code (je vois que t'es en chien d'avoir la réponse sur un plateau d'argent), je t'explique un peu le principe du bouzin. C'est pas compliqué, je vais juste répéter ce que je disais plus haut.

Bon alors, grosso-modo, on va commencer par avoir un état courrant (*current_state*, j'ai eu le TOEIC ma gueule) qui va avoir la valeur `0` (notre état initial). Ensuite, avec notre chaîne de caractèren on doit voir si elle _match_ ou pas. On va donc entrer dans une boucle qui va itérer dans celle ci et on va modifier notre *current_state* pour qu'il obtienne le prochain état selon le caractère courrant. Cela ressemble un peu à ça:

{% highlight c %}
int 	to_alphabet(char c)
{
	switch (c)
	{
		case 'd': return (0);
		case 'i': return (1);
		case 'n': return (2);
		case '2': return (3);
		case 'o': return (4);
		case 'b': return (5);
		case 'a': return (6);
		default: return (7);
	}
}

void	lexer(std::string str)
{
	int i = 0;
	int current_state = 0;

	while (i < str.size())
	{
		current_state = t[current_state][to_alphabet(str[i])];
		std::cout "Je suis à l'état: " << current_state << std::endl;
		i  = i + 1;
	}
}
{% endhighlight %}

{% highlight sh %}
$ ./a.out "din"
Je suis à l'état 1
Je suis à l'état 2
Je suis à l'état 3
{% endhighlight %}

{% highlight sh %}
$ ./a.out "did"
Je suis à l'état 1
Je suis à l'état 2
Je suis à l'état 0
{% endhighlight %}

Bon tu vois maintenant le principe de passage d'un état à un autre maintenant. C'est grâce à notre matrice de transition. Mais on utilise pas encore notre matrice des actions, il le faut pourtant pour voir quand notre chaîne _match_ avec notre expression régulière. Donc on va étoffer un peu plus le code pour exécuter les actions qu'on a décrites plus haut en fonction de l'état et du caractère.

{% highlight c %}
void	lexer(std::string str)
{
	int i = 0;
	int current_state = 0;
	std::string token;

	while (i < str.size())
	{
		switch (a[current_state][to_alphabet(str[i])])
		{
			case MA:
				token += str[i];
				current_state = t[current_state][to_alphabet(str[i])];
				break;
			case HR:
				std::cout << token << " match !" << std::endl;
				token.clear();
				current_state = 0;
				break;
			case ER:
				token.clear();
				current_state = 0;
				break;
		}

		i = i + 1;
	}

	if (a[current_state][to_alphabet(str[i])] == HR)
		std::cout << token << " match !" << std::endl;
}
{% endhighlight %}

Bon on a un truc qui marche pas trop mais calme toi un peu c'est pas encore fini. On a un dernier problème. Si tu donnes la chaîne `din2oba`, c'est cool, ça marche. Si tu passes la chaîne `din2obaLOLdin2oba`, ça _match_ deux fois, c'est cool, ça marche. Par contre, si tu passes `din2obadin2oba`, bah là, ça _match_ qu'une fois ! Le problème réside dans l'action `HR`. En faite, quand on tombe sur l'action `HR`, on a consumé toute la première occurence de `din2oba` et le `d` de la deuxième occurence pour ensuite passer à l'état `0` (au lieu du `1`). Et c'est ça qui fait planter parce que, ça se trouve, après être dans l'action `HR`, il peut y avoir une autre occurrence comme dans le dernier exemple, il ne faudrait pas consumer ce dernier caractère `d` mais le laisser pour une nouvelle itération.

{% highlight c %}
void	lexer(std::string str)
{
	int i = 0;
	int current_state = 0;
	std::string token;

	while (i < str.size())
	{
		switch (a[current_state][to_alphabet(str[i])])
		{
			case MA:
				token += str[i];
				current_state = t[current_state][to_alphabet(str[i])];
				break;
			case HR:
				if (t[0][to_alphabet(str[i])] != current_state)
				{
					std::cout << token << " match !" << std::endl;
					token.clear();
				}
				if (a[0][to_alphabet(str[i])] != ER)
					i = i - 1;
				else
					token += str[i];

				current_state = 0;
				break;
			case ER:
				token.clear();

				if (a[0][to_alphabet(str[i])] != ER)
					i = i - 1;

				current_state = 0;
				break;
		}

		i = i + 1;
	}

	if (a[current_state][to_alphabet(str[i])] == HR)
		std::cout << token << " match !" << std::endl;
}
{% endhighlight %}

Bon voilà, j'ai juste fait un retour en arrière. Si passer de l'état `0` à un autre état avec le caractère courant ne produit pas l'action `ER`. Cela veut dire qu'on a hypothétiquement une autre occurence juste après. Il faut donc _sauvegarder_ le caractère. Ensuite, j'ai rendu plus spécifique l'apparation du _match_. Si on tombe sur ce genre d'expression `din2ob(a)+` (ça voudrait dire que depuis le `HR`, on puisse tomber sur un état dont l'action est `MA`, ou plus visuellement, notre état final aurait une transition pointant sur lui même), au lieu de délivrer le token quand nous sommes fasses à un `HR`, il faut voir si l'expression régulière ne continue pas. Si c'est le cas, il faut rajouter le caractère courant au _token_ et itérer une nouvelle fois. Sinon, c'est que nous sommes vraiment à la fin et nous pouvons dire qu'on a trouvé une occurrence.

![MTHRFCKR Cas]({{ site.baseurl }}/img/05.png)

## Conclusion

Bon t'as réussi à implémenter un petit automate à état fini de merde. Bah oui, là t'as fait un truc tellement con qu'il existe déjà des logiciels qui te génère ce bouzin à ta place. Mais ça c'est pas fait en un jour et j'espère que t'as bien capter le principe parce que la prochaine fois, on va aller à un autre niveau. On va faire des graphes et tout et tout, t'inquiètes.

Tu comprendras que ici, on était plus ou moins spécifique aux expressions régulières mais tu peux très bien faire en sorte que tes transitions dépendent de autre chose que des caractères et que tes actions fassent autre chose que rajouter ces mêmes caractères à un _token_. Les automates à états finis sont dans d'autres domaines comme le réseau ou l'I.A. donc rentre toi bien ça dans le crâne. C'est pasun outil kikoo, c'est bien un truc qu'on utilise tout les jours.