### 3.4 Organisation des classes

Pour analyser l’organisation des classes, je me suis appuyé sur les classes compilées présentes dans les répertoires target/classes et target/test-classes.
Au total, j’ai identifié 264 classes uniques.

### 3.4.1 Analyse de la hiérarchie

J’ai utilisé deux indicateurs classiques :

DIT (Depth of Inheritance Tree)
NOC (Number of Children)
Résultats obtenus

DIT min = 0
DIT max = 2
DIT moyenne = 0,08

La profondeur d’héritage est donc très faible. La grande majorité des classes n’utilisent pas de hiérarchie complexe. On ne trouve pas de chaînes d’héritage longues, ce qui simplifie la compréhension globale du projet.

Concernant le NOC :
NOC min = 1
NOC max = 200
NOC moyenne = 15,87

La valeur maximale élevée indique qu’au moins une classe joue un rôle très central, avec de nombreux descendants. Cela correspond généralement à une interface ou une classe abstraite structurante.

Globalement, la hiérarchie est plate, ce qui limite les effets de dépendances en cascade liés à l’héritage.

### 3.4.2 Stabilité et couplage

L’analyse jdeps du JAR gson met en évidence 2414 dépendances entre classes.
Ce volume important de dépendances s’explique par :
L’utilisation intensive de la réflexion Java
La dépendance aux packages standards (java.lang, java.util, java.io, java.lang.reflect)
Le rôle central de la bibliothèque dans la sérialisation

Le couplage est donc dense, mais cohérent avec la nature d’un moteur de sérialisation. Certaines classes apparaissent particulièrement structurantes, car elles sont utilisées par de nombreux autres composants.

### 3.4.3 Analyse de la cohésion

J’ai mesuré la cohésion interne de trois paquetages représentatifs :
- com.google.gson : cohésion = 0,422
- com.google.gson.internal : cohésion = 0,20
- com.google.gson.internal.bind : cohésion = 0,171

Le paquetage principal (com.google.gson) présente la meilleure cohésion. Les classes qui le composent collaborent davantage entre elles.

Les paquetages internes ont une cohésion plus faible. Cela s’explique par leur rôle technique : ils servent d’infrastructure et interagissent avec plusieurs autres composants (adaptateurs, lecteurs, writers, réflexion).

Cette différence est cohérente avec une architecture en couches.

### 3.4.4 Conclusion
L’organisation des classes montre :
- Une hiérarchie peu profonde
- Un noyau central avec une classe fortement structurante
- Un couplage important mais cohérent avec le rôle du projet
- Une cohésion plus forte au niveau de l’API publique que dans les couches internes
L’architecture globale est donc cohérente avec celle d’une bibliothèque de sérialisation modulaire, centralisée autour d’un moteur interne solide.