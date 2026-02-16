# 3.1  Utilisation de bibliothèques extérieures

Q1 – Utilisation de bibliothèques extérieures
Le projet est structuré en 8 modules Maven.
L’analyse de l’ensemble des fichiers pom.xml montre :
36 dépendances déclarées au total (tous modules confondus, doublons inclus)
17 bibliothèques extérieures uniques identifiées par le couple groupId:artifactId
Ces dépendances concernent principalement :
des bibliothèques de tests (JUnit, Truth),
des bibliothèques utilitaires (Guava),
et des bibliothèques spécialisées selon les modules (Jackson, Protobuf, Caliper, ProGuard).

Le projet s’appuie donc sur un nombre limité et maîtrisé de bibliothèques extérieures, réutilisées de manière cohérente entre les modules.