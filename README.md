# dhbw-shader
Dieser GLSL-Shader rendert das DHBW-Logo, modelliert 3-dimensionales Objekt. Der Shader wurde als Prüfungsleistung für den Kurs Computergrafik an der DHBW Stuttgart erstellt.

# Dokumentation
## Einleitung
### Problemstellung und Zielsetzung
Die vorliegende Arbeit besch¨aftigt sich mit der Implementierung eines OpenGL Shading Language (GLSL)-Fragmentshaders. Ziel ist es, das Duale Hochschule BadenWurttemberg (DHBW)-Logo allein durch mathematische Formeln a ¨ ls eine computergenerierte Grafik im 3-dimensionalen (3D)-Raum darzustellen. Die generierte
Szene wird durch zus¨atzliche Implementierungen wie Boden, Beleuchtung, Schatten,
Kameraperspektive, Nebel, etc. komplementiert. Durch die Integration der zus¨atzlichen
grafischen Komponenten wird ein besseres Verst¨andnis fur die 3-dimensionale An- ¨
ordnung der Szene geschaffen und der Beobachter bei der Tiefenwahrnehmung unterstutzt. ¨
1.2 Methodische Vorgehensweise
Um einen m¨oglichst einfachen Einstieg in die Modellierung 3-dimensionaler Objekte
in GLSL zu erhalten, wird auf das bestehende Projekt ”
Greek Temple“von Inigo
Quilez von der Webseite ”
Shadertoy“zuruckgegriffen. Es implementiert bereits eine ¨
3D-Umgebung, die im Verlauf des Projekts verwendet wird. Das Projekt ist unter
dem Link Shadertoy - Greek Temple zu finden. Im bestehenden Projekt werden
alle nicht ben¨otigten Funktionen, wie beispielsweise die Modellierung des griechischen Tempels und der Umgebungslandschaft, entfernt. Anschließende wird mit der
3D-Modellierung des DHBW-Logos anhand mathematischer Prinzipien fortgefahren. Zuletzt werden die zus¨atzlichen grafischen Komponenten integriert und speziell
fur die modellierte Szene angepasst. ¨
Im vorliegenden Dokument sind alle blau-markierten Links und Querverweise klickbar.

## Funktionsweise des Shaders
### Inbetriebnahme
Die Inbetriebnahme des Shaders gestaltet sich ¨außerst simpel: Der beiliegende GLSLCode kann einfach in ein neues Projekt auf der Shadertoy-Webseite kopiert werden.
Die integrierte WebGL-Umgebung von Shadertoy rendert den eingefugten Code ganz ¨
ohne Anpassungen durch Klick auf ”
Kompilieren“ im nebenstehenden Fenster.
### Verwendete mathematische Prinzipien
In diesem Kapitel werden die grundlegenden mathematischen Prinzipien fur die ¨
Modellierung der Szene und letztlich fur die Implementierung des Shaders erl ¨ ¨autert.
Die entstehenden grafischen Objekte sind die Grundlage fur die in Kapitel ¨ 2.3 und
### gezeigten Implementierungen.
#### Kugel
Um das Prinzip der Modellierung mathematischer Objeke einmal darzustellen, wird
mit der einfachsten Modellierung eines Objekts begonnen: der Kugel. Die Kugel
kann durch die mathematische Gleichung
x
2 + y
2 + z
2 = r
2
beschrieben werden, wobei x, y und z die 3D-Koordinaten im Raum und r den
Radius der Kugel repr¨asentiert. Um die Kugel leichter modellieren zu k¨onnen, wird
die Formel zu
p
x
2 + y
2 + z
2 − r = d
umgestellt. Der Parameter d stellt nun den Output der Formel dar, der in Abh¨angigkeit
vom Radius die Kugel im Raum darstellt. Wird die Formel mittels GLSL implementiert, ergeben sich folgende Codezeilen:
1 vec3 pos = vec3 (0.0 , 0.0 , 0.0) ;
2 float d = length ( pos ) - 1.0;
Mit diesem Code wird eine Kugel mit Radius 1.0 L¨angeneinheiten (LE) im Ursprung
erstellt. (Siehe Abbildung 1.)
Abbildung 1: Modellierte Kugel im Raum
Die Funktion length() quadriert dafur die einzelnen x-, y- und z-Komponenten und ¨
zieht die Wurzel aus ihrer Summe.

#### Box
Eine Box bzw. ein Quader im 3D-Raum wird durch eine sogenannte Signed-DistanceFunction (SDF) modelliert. SDFs werden h¨aufig in der Computergrafik zur Modellierung 2-dimensionaler und 3-dimensionaler Objekte eingesetzt. Wie der Name vermuten l¨asst, wird hiermit der Abstand eines Punkts im Raum zum entsprechenden
Objekt gemessen. Ist der Punkt außerhalb des Objekts, gibt die SDF die euklidische
Distanz zur Oberfl¨ache des Objekts als positiven Wert zuruck. Enth ¨ ¨alt das Objekt
den eingegebenen Punkt, ist die ausgegebene Distanz negativ. Auf der Oberfl¨ache
des Objekts ergibt sich ein Funktionswert von 0. Die folgenden Code-Zeil den zeigen
die implementierte SDF eines Quaders in GLSL:
1 float sdBox ( vec3 pos , vec3 b )
2 {
3 vec3 d = abs ( pos ) - b ;
4 return min ( max ( d .x , max ( d .y , d . z ) ) ,0.0) + length ( max (d
,0.0) ) ;
5 }
Der Eingabeparameter pos repr¨asentiert wieder die Position im Raum, 3D-Vektor
b enth¨alt die Halbdimensionen des Quaders. Die SDF ermittelt die Differenz d zwischen der absoluten Position p und den Halbdimensionen, um den Abstand zum
Quader zu bestimmen. Die Funktion wird beispielsweise folgendermaßen aufgerufen:
1 float d = sdBox ( pos , vec3 (0.5 , 0.5 , 0.5) ) ;
Hier wird an der Stelle pos ein Quader mit den Kantenl¨angen von 1 LE erzeugt.
Der sich ergebende 1LE3 große Wurfel ist in Abbildung ¨ 2 zu sehen:
Abbildung 2: Modellierter Wurfel im Raum ¨
Dieser Quader bildet nachfolgend die Basis fur alle Modellierungen von rechteckigen ¨
Wurfeln, Boxen und Quadern. ¨

#### Halb-Ellipse
Zur Modellierung von runden Kanten im DHBW-Logo wird noch eine weitere BasisGeometrie ben¨otigt. Die Halb-Ellipse wird ebenfalls durch eine SDF dargestellt:
1 float sdHalfEllip ( vec2 pos , vec2 size )
2 {
3 pos . x = max (0.0 , pos . x ) ;
4 float smallestRadius = min ( size .x , size . y ) ;
5 return ( length ( pos / size ) * smallestRadius ) -
smallestRadius ;
6 }
Als Eingabeparameter finden sich mit pos wieder die Position im 2D-Raum und mit
size die beiden Halbachsen der Ellipse. Zun¨achst wird die x-Koordinate der Position
auf nicht-negative Werte beschr¨ankt, um die Ellipse zu halbieren. Anschließend wird
die kleinere der beiden Halbachsen als Referenzradius bestimmt. Der Punkt p wird
durch die Gr¨oße der Ellipse skaliert, seine normierte L¨ange berechnet und mit dem
Referenzradius verrechnet. Zu beachten ist, dass die SDF der Halb-Ellipse nur in der
2-dimensionalen Fl¨ache arbeitet. Wurde die Funktion auf dem 3D-Raum erweitert, ¨
wurde eine Art Kugel mit Abrundungen in z-Richtung entstehen. D ¨ a das DHBWLogo sp¨ater im Profil modelliert werden soll ist die Abrundung in z-Richtung nicht
erwunscht und die Modellierung der betroffenen Buchstaben erfo ¨ lgt uber den 2- ¨
dimensionalen Weg.

### Aufbau der Szene
Dieses Kapitel besch¨aftigt sich mit der Modellierung des 3-dimensionalen DHBWLogos aus den in Kapitel 2.2 beschriebenen Modellierungselementen. Zu Ubersichtszwecken ¨
werden Kommentare aus den Quellcodeausschnitten entfernt. Sie sind jedoch im beiliegenden Originalcode weiterhin enthalten.
#### Buchstabe ”
D“
Der Buchstabe ”
D“setzt sich aus einem Quader und zwei Halb-Ellipsen zusammen.
Die innere Halb-Ellipse wird dabei von der ¨außeren Halb-Ellipse subtrahiert. Das Ergebnis wird durch einen Quader nach links hin begrenzt. Der vollst¨andige Quellcode
zur Modellierung des Buchstaben ”
D“ist nachfolgend dargestellt:
1 float Dhbw ( vec3 p , float thick )
2 {
3 p . x += 15.0;
4
5 float d = sdHalfEllip ( p . xy , vec2 (5.0 , 6.0) ) ;
6 d = max (d , -p . x - 3.5) ;
7 float d2 = sdHalfEllip ( p . xy , vec2 (3.0 , 4.0) ) ;
8 d2 = max ( d2 , min ( - p . y + 5.0 , -p . x - 1.5) ) ;
9 d = max (d , - d2 ) ;
10 float Dhbw = max (d , abs ( p . z ) - thick ) ;
11
12 return Dhbw ;
13 }
Zun¨achst wird die Position des Buchstabens in x-Richtung angepasst, sodass alle
Buchstaben nebeneinander Platz finden. Im Anschluss werden an verschiedenen Positionen die Halb-Ellipsen definiert, welche voneinander subtrahiert werden. In Zeile
10 wird manuell der nach links begrenzende Quader definiert. Abschließend wird die
Modellierung als Floating-Point-Variable ausgegeben. Wird der Code durch Aufruf
der Funktion (siehe Kapitel 2.3.5) in WebGL gerendert, entsteht folgendes Bild:
Abbildung 3: Modellierter Buchstabe ”
D“
#### Buchstabe ”
H“
Der Buchstabe ”
H“wird rein durch verschieden positionierte Quader modelliert.
Hierfur kommt die in Kapitel ¨ 2.2.2 aufgezeigte SDF einer Box bzw. eines Quaders
zum Einsatz. Der GLSL-Code ist nachfolgend dargestellt:
7
1 float dHbw ( vec3 p , float thick )
2 {
3 p . x += 5.0;
4
5 vec3 Hlpos = vec3 ( p . x +3.0 , p .y , p . z ) ;
6 float Hl = sdBox ( Hlpos , vec3 (1.0 ,6.0 , thick ) ) ;
7 vec3 Hrpos = vec3 ( p .x -3.0 , p .y , p . z ) ;
8 float Hr = sdBox ( Hrpos , vec3 (1.0 ,6.0 , thick ) ) ;
9 vec3 Hmpos = vec3 ( p .x , p .y , p . z ) ;
10 float Hm = sdBox ( Hmpos , vec3 (3.0 ,1.0 , thick ) ) ;
11
12 float dHbw = min ( Hl , Hr ) ;
13 dHbw = min ( dHbw , Hm ) ;
14 return dHbw ;
15 }
Auch hier wird der gesamte Buchstabe auf seine korrekte Position in x-Richtung
geruckt. Danach werden die einzelnen Quader mit ihren korrekte ¨ n Dimensionen und
Positionen erzeugt. Sie werden abschließend in Zeile 12 und 13 durch entsprechende
Min-Operationen zusammen zu einer Ausgabe gefugt. Das Ergebnis wird wiederum ¨
als FLoating-Point-Variable ausgegeben.
#### Buchstabe ”
B“
Das ”
B“wird analog zum ”
D“mithilfe der SDFs von Boxen und Halb-Ellipsen erzeugt. Der Quellcode ist hier aus Platzgrunden nicht eingef ¨ ugt, ist jedoch im bei- ¨
liegenden Originalcode einsehbar. Die Schwierigkeit beim ”
B“besteht darin, dass es
nicht mithilfe zweier Instanzen von ”
D“modelliert werden kann, da hier die Proportionen deutlich voneinander abweichen. Zudem wird der obere Halbbogen etwas kleiner als der untere Halbbogen modelliert, sodass beide Teile getrennt voneinander implementiert werden. Die erstellten Teile werden am Ende wieder mit
min-Funktionen zusammengefugt und das Ergebnis ausgegeben. Es ist nach dem ¨
Rendering in Abbildung 4 zu sehen:

#### Buchstabe ”W“
Um den Modellierungsprozess des Buchstabens ”
W“zu erleichtern, wird er mithilfe
zweier ”
V“modelliert. Der Quellcode wird hier ebenfalls aus Platzgrunden nur teil- ¨
weise dargestellt. Innerhalb der Modellierungsfunktion des ”
V“wird mithilfe einer
Normalisierungsfunktion ein Vektor auf die L¨ange 0 LE bis 1 LE normalisiert. Dadurch beh¨alt der Vektor seine Richtung, jedoch hat seine L¨ange keinen Einfluss auf
das nachfolgend gebildete Skalarprodukt mit den x- und y-Koordinaten der Position
in der Fl¨ache.
1 float dhbV ( vec3 p , float thick )
2 {
3 [...]
4 vec2 a = normalize ( vec2 (10.0 , -2.5) ) ;
5 float v = dot ( p . xy , a ) ;
6 [...]
7 }
Durch den Richtungsvergleich der beiden Vektoren durch das Skalarprodukt wird
eine Schr¨age generiert, die als Flugel des ¨
”
V“dienen. Subtrahiert man zwei ineinander
liegende und in y-Richtung verschobene V-f¨ormige Objekte voneinander, wird ein
”
V“erzeugt. In der Funktion dhbW werden zwei in x-Richtung nebeneinanderstehende
”
V“miteinander verknupft, wodurch sich ein ¨
”
W“ergibt. Fur eine ansprechendere ¨
Optik werden die beiden unteren Spitzen des ”
W“durch die Begrenzungsfl¨ache
1 v = max (v , -p . y + 3.0) ;
abgeschnitten.

Die finale Modellierung des Buchstaben ”
W“ist in Abbildung 6 dargestellt:
Abbildung 6: Modellierter Buchstabe ”
W“
#### Gesamte Szene mit Boden
Final wird die komplett modellierte Szene zusammengebaut: Hierfur werden zun ¨ ¨achst
die einzelnen Instanzen der Buchstaben erzeugt und anschließen mittels min-Funktionen
miteinander verknupft: ¨
1 vec3 scene ( in vec3 p )
2 {
3 p . y += 3.0; // Anheben der Position
4 float thickness = 7.0;
5
6 // Buchstaben erzeugen
7 float Dhbw = Dhbw (p , thickness ) ;
8 float dHbw = dHbw (p , thickness ) ;
9 float dhBw = dhBw (p , thickness ) ;
10 float dhbW = dhbW (p , thickness ) ;
11
12 // Buchstaben darstellen
13 float DHbw = min ( Dhbw , dHbw ) ;
14 float DHBw = min ( DHbw , dhBw ) ;
15 float DHBW = min ( DHBw , dhbW ) ;
16
17 return vec3 ( DHBW , 1.0 , 0.5) ;
18 }
Zu beachten ist hierbei, dass die gesamte Szene um 3 LE in y-Richtung angehoben wird, um einen Schwebeeffekt fur das Logo zu erzeugen. Der Effekt kann f ¨ ur ¨
das gesamte Logo je nach Bedarf durch die Anderung einer einzelnen Variablen ¨
ver¨andert oder sogar entfernt werden. Zudem ist in der finalen Modellierung die Variable thickness integriert, die beim Aufruf aller Buchstabenfunktionen ubergeben ¨
wird. Durch sie kann die Ausdehnung in z-Richtung fur das gesamte Logo an einer ¨
Stelle im Code angepasst werden. Innerhalb der Funktion map wird die Modellierung
des DHBW-Logos aufgerufen. Zudem wird die Bodenfl¨ache fur die gesamte Szene ¨
auf einer festgelegten H¨ohe in y-Richtung generiert und das Gesamtergebnis der
Modellierung ausgegeben:
1 vec3 map ( in vec3 p )
2 {
3 vec3 res = scene ( p ) ; // Erzeugung von " DHBW "
4 float m = p . y + 9.5; // Bodenhoehe
10
5 if(m < res . x ) res = vec3 (m , 3.0 , 1.0) ; // Erzeugung des
Bodens
6 return res ;
7 }
Nach dem Rendering der Modellierung ergibt sich folgendes Ergebnis in Abbildung
7:

### Weitere grafische Elemente
Bei der Implementierung sonstiger grafischer Elemente wurde sich an den bereits
bestehenden Implementierungen des verwendeten Projekts orientiert. Im Folgenden
wird die Funktionsweise der auf die Szenerie angepassten und implementierten Grafikelemente rudiment¨ar beschrieben. Leider kann hier aus Platzgrunden nicht auf ¨
alle weiteren grafischen Elemente eingegangen werden.
2.4.1 Beleuchtung
Fur die Beleuchtung sind vornehmlich drei Lichtquellen zust ¨ ¨andig. Als Basis dient
das Sonnenlicht, dessen Richtung am Anfang des Quellcodes durch
1 vec3 sunLig = normalize ( vec3 (0.7 ,0.1 ,0.4) ) ;
definiert wird. Es wird auch als Key-Light bezeichnet, da es die Hauptlichtquelle zur
Beleuchtung der Szene darstellt.
1 vec3 col = vec3 (0.0) ;
2 col += amb *1.0* vec3 (0.15 ,0.25 ,0.35) * occ *(1.0+ mter ) ; //
Ambient - Light
3 col += dif *5.0* vec3 (0.90 ,0.55 ,0.35) ; // Key - Light
4 col += bak *1.7* vec3 (0.10 ,0.11 ,0.12) * occ * mbou ; // Back -
Light = Reflektiertes Licht der Sonne ( in
entgegengesetzter Richtung )
Neben dem Key-Light wird das Ambient-Licht verwendet, um das blaue Licht des
Himmels zu simulieren. Das Ambient-Licht ist entsprechend realer Gegebenheiten
ungef¨ahr um Faktor 10 schw¨acher als das Key-Light. Seine Lichtfarbe passt gut zum
Key-Light, da blau und gelb auf der Farbskala komplement¨are Farben darstellen.
Zuletzt wird das Back-Light verwendet, um reflektierende Sonnenstrahlen in entgegengesetzter Richtung (also in den Schattenseiten) zu simulieren. Zu beachten ist
11
zudem, dass das Ambient- und das Back-Light mit dem Faktor occ multipliziert werden. Dieser Faktor repr¨asentiert den Occlusion-Faktor. Occclusion wird verwendet,
um realistische Licht- und Schattendarstellungen zu erzeugen, indem berucksichtigt ¨
wird, welche Teile einer Szene oder eines Objekts durch andere verdeckt werden.
Durch diese drei Beleuchtungsquellen wird die gesamte Szene optimal ausgeleuchtet.
#### Himmel
Die Farbe des Himmels wird in einer weiteren ausgelagerten Funktion erzeugt:
1 vec3 skyColor ( in vec3 ro , in vec3 rd )
2 {
3 vec3 col = vec3 (0.3 ,0.4 ,0.5) *0.3 - 0.3* rd . y ; // blau
4 col = mix ( col , vec3 (0.2 ,0.25 ,0.30) *0.5 , exp ( -30.0* rd . y
) ) ; // grau
5 float sd = pow ( clamp ( 0.25 + 0.75* dot ( sunLig , rd ) , 0.0 ,
1.0) , 4.0) ; // rot
6 col = mix ( col , vec3 (1.2 ,0.30 ,0.05) /1.2 , sd * exp ( - abs
((60.0 -50.0* sd ) * rd . y ) ) ) ;
7 return col ;
8 }
Zun¨achst wird der Himmel blau eingef¨arbt. Je nach vertikaler Position im Himmel
werden Pixel, die sich n¨aher am Horizont befinden, zunehmend grau eingef¨arbt.
Dies wird durch die mix-Funktion sichergestellt, die die eingegebenen Farben in yRichtung interpoliert. Die in die mix-Funktion eingegebene Exponentialfunktion ist
deshalb von der y-Koordinate abh¨angig. In Richtung Sonne wird der Himmel dann
zunehmend rot eingef¨arbt, wie am Skalarprodukt mit dem Sonnenlicht-Vektor zu
erkennen ist.
#### Nebel
Der nachfolgende Code zeigt die angepasste Implementierung des Nebel-Effekts am
Horizont:
1 vec3 fogcol = vec3 (0.1 ,0.125 ,0.15) ; // grau
2 float sd = pow ( clamp ( 0.25 + 0.75* dot ( lig , rd ) , 0.0 , 1.0) ,
4.0) ; // rot
3 fogcol = mix ( fogcol , vec3 (1.0 ,0.25 ,0.042) , sd * exp ( - abs
((60.0 -50.0* sd ) * abs ( rd . y ) ) ) ) ;
4 float fog = 1.0 - exp ( -0.0013* t ) ; // Entfernung
5 col *= 1.0 -0.5* fog ; // geringere I n t e n s i t t
6 col = mix ( col , fogcol , fog ) ;
Zun¨achst wird eine graue Farbe des Nebels festgelegt, in die bei zunehmender N¨ahe
zum Sonnenlicht (lig) ein roter Farbton eingemischt wird. Bei zunehmender Entfernung in y-Richtung vom horizont wird der Effekt durch Zeile 5 abgeschw¨acht.
Zuletzt wird die Himmelsfarbe mit der Nebelfarbe und der entsprechenden Distanz
von Horizont interpoliert, wodurch sich ein gleichm¨aßig verteilter Nebeleffekt um
den Horizont herum ergibt.
#### Kamera
Auch die Kameraplatzierung und -winkel kann ganz genau an die Szenerie angepasst
werden: Die n¨otigen Einstellungen der Parameter werden dafur in der ¨ mainImageFunktion getroffen:
1 float an = -7.0 + sin ( iTime *0.15) *2.0; // Kameraschwenk (
Seite , Geschwindigket , Weite )
2 float ra = 60.0; // Kameraentfernung
3 float fl = 3.0; // Kamerazoom
4 vec3 ta = vec3 (0.0 , -3.0 ,0.0) ; // Kamerposition
5 vec3 ro = ta + vec3 ( ra * sin ( an ) ,12.0 , ra * cos ( an ) ) ; //
Kamerahoehenwinkel
6 mat3 ca = setCamera ( ro , ta , 0.0) ; // Kameraseitenwinkel
Zu beachten ist, dass beim Kameraschwenk das uniform iTime ein multipliziert wird.
Es handelt sich dabei um eine von der CPU an den Fragment-Shader ubergebene ¨
Variable, die aktuelle Zeit repr¨asentiert. Somit wird die Darstellung der gesamten Szene zeitlich abh¨angig gemacht, was in einem Kameraschwenk von links nach
rechts und vice versa um das DHBW-Logo resultiert. Geschwindigkeit, Winkel und
Schwenkweite des Kameraschwenks k¨onnen dabei ebenso angepasst werden, wie die
Entfernung der Kamera zur Szene, der Winkel in x- und y-Richtung usw.
Durch die Kombination der angepassten Effekte, wie Beleuchtung, Himmel und weiteren Effekten, entstehen optisch sehr ansprechende Szenerien, wie in Abbildung 8
zu sehen:
Abbildung 8: Modelliertes DHBW-Logo mit optischen Effekte

## Fazit und Ausblick
Im Rahmen des Projekts wurde ein GLSL-Shader entwickelt, der das DHBW-Logo
in einer 3-dimensionalen Umgebung darstellt. Alle der in Kapitel 1.1 gesetzten Ziele
konnten erfullt werden. Das Projekt kann somit als voller Erfolg betrac ¨ htet werden.
Durch einige Anpassungen kann die modellierte Szene sogar wieder in die Landschaftsumgebung des originalen GLSL-Projekts gesetzt werden. Durch die Integration einiger externer Texturen, die uber sogenannte ¨
”
iChannel“in Shadertoy eingespielt werden, ergibt sich ein optisch sehr ansprechendes Bild (siehe Abbildungen 9
und 10), das auch auf dem Deckblatt der vorliegenden Dokumentation zu sehen ist:
Das Projekt kann in vielerlei Hinsicht noch erweitert werden: Beispielsweise kann der
Quellcode in eine unabh¨angige HTML-Datei portiert werden, wodurch die WebGLLaufzeitumgebung des Browsers genutzt wird, um unabh¨angig von Shadertoy zu
sein. Innerhalb des Shaders k¨onnen beispielsweise die Farben und Texturen der
Buchstaben angepasst werden, um ein Design des DHBW-Logos zu erzeugen, das
dem real verwendeten Farbmuster entspricht.
