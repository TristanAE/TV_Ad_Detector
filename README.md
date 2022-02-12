# TV_Ad_Detector
 Traite image TV pour mettre en muet si pub detectée

# I-	Conception électronique 


a)	Diffusion du flux vidéo sur l’ordinateur


Tout d’abord l’objectif est de récupérer le flux vidéo permettant d’avoir les chaines TV. 
Je me sers dans mon cas de mon boitier TV Orange dont je récupère le flux directement grâce son câble HDMI. Il faut ensuite pouvoir envoyer ce flux vers l’ordinateur, je vais donc utiliser une carte d’acquisition HDMI 4k USB.
Dans un second temps, nous souhaitons obtenir à la fois le flux vidéo sur notre téléviseur et sur notre écran d’ordinateur. Nous devons donc nous procurer un splitter : Une entrée HDMI vers 2 sorties HDMI.
L’unique entrée est branchée au boitier Orange, une des sorties au téléviseur et enfin la dernière à la carte d’acquisition puis à l’ordinateur.

b)	Emetteur infrarouge


Pour interagir avec notre télévision, nous aurons besoin d’une télécommande que l’Arduino puisse contrôler soit un émetteur infrarouge.
Cet émetteur est connecté à l’Arduino en le branchant au GND et à un pin digital.


# II-	Conception informatique


Pour pouvoir détecter nos publicités, nous allons nous servir du logo de la chaine. Nous pouvons remarquer que lors d’une émission ce logo apparait dans le coin en haut à droite, puis disparait pendant les publicités.
Pour le moment, la seule chaine que nous essaierons d’analyser est France 3. Il faudra d’abord entrainer notre programme à reconnaitre son logo puis nous traiterons l’image reçue pour isoler ce dernier. Enfin nous pourrons dire si nous visionnons une pub ou une émission et actionner notre émetteur.


a)	Entrainement pour la détection


Pour que notre programme puisse reconnaitre le logo nous allons utiliser un logiciel adapté. Nous lui implémentons une cinquantaine d’images positives et négatives, avec et sans le logo.  Une fois le traitement terminé, nous obtenons un fichier XML que nous utiliserons avec OpenCV.


b)	Traitement du flux 


La première chose à réaliser sur OpenCV est la capture du flux vidéo. Les chaines TV Orange nous arrivent via la carte d’acquisition, c’est pourquoi nous mettrons le port sur 1.
Nous allons ensuite centrer l’image pour n’avoir à analyser que la zone où le logo pourrait se trouver (et ainsi enlever les chances de détecter d’autres objets ressemblant à celui que l’on cherche). 
Enfin nous transformerons le « morceau » d’image en nuance de gris.
Les paramètres implémentés dans threshold() sont les valeurs 230 et 255 : chaque pixel inferieur à 230 (soit presque toutes les couleurs sauf le blanc) devient blanc et les pixels supérieurs deviennent noirs. 
Ainsi la couleur blanche du logo devient noire et son environnement devient blanc, ce qui facilite grandement sa détection.


c)	Détection pub ou émission


Haar cascade :
Nous devons dans cette partie utiliser le fichier Haar cascade que nous avons fabriqué précédemment.
Nous allons donc tout d’abord déclarer une variable ayant accès à toutes les données de notre fichier en passant par la méthode CascadeClassifier() : 

Ensuite, après avoir traité notre image, nous déclarons une seconde variable et nous appelons la méthode detectMultiScale(). Cette méthode va se servir de la variable contenant toutes les données du fichier Xml pour analyser l’image.

Ainsi dès que le logo est détecté, la deuxième variable, qui est un tableau, devient non nulle.


Détection des pubs :


Nous allons utiliser la bibliothèque time pour permettre au programme d’analyser pendant 15 secondes l’image puis d’attendre à nouveau 15 secondes pour à nouveau lancer une analyse. 
Pourquoi 15 secondes ?

-15 secondes de battement permettent une vérification fréquente pour ne pas perdre plusieurs minutes devant une pub

-15 secondes d’analyse permettent de donner assez de temps pour que les plans de la caméra changent si jamais le logo est mal détecté


Dès que le temps d’attente de 15 secondes se termine, la vérification peut commencer :

A chaque détection, un compteur s’incrémente pendant 15 autres secondes. A la fin du temps imparti, si le compteur est supérieur à 50 alors nous sommes sûrs que le logo est présent et nous pouvons dire que nous regardons une émission. Dans le cas contraire, alors une pub est jouée.

d)	Rendre muet le son

Pour envoyer une information de Python vers Arduino, nous utiliserons la bibliothèque Serial. Pour envoyer un code hexadécimal avec l’émetteur, nous utiliserons la bibliothèque IRremote sur Arduino.

A chaque pub détectée, nous envoyons « 1 » à l’Arduino et sinon « 0 » en binaire .
Du côté de l’Arduino, un programme lui permet de récupérer ces données venant du port serial puis en fonction de la valeur, d’agir.
Que le caractère soit 1 ou 0, le code hexadécimal correspondant au bouton muet de la télécommande est envoyé. Ce sont des variables blindant le programme qui permettent de conserver le son muet si plusieurs pubs sont détectées successivement ou au contraire de garder le son actif durant une émission.

Remarque : Pour obtenir le code hexadécimal du bouton muet d’une télécommande, nous avons testé préalablement le signal émis par celle-ci avec un capteur infrarouge.


# III-	Rendu final

![ezgif com-gif-maker (4)](https://user-images.githubusercontent.com/92324336/153722804-fcec1526-7dc3-484a-8d9e-72303da4d81e.gif)

