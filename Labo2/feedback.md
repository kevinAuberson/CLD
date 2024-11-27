# Feedback - CLD lab 2 - App scaling on IaaS

## Groupe
* GrR
* Kevin Auberson
* Léo Zmoos

## TASK 1: CREATE A DATABASE USING THE RELATIONAL DATABASE SERVICE (RDS)

* Rien à signaler.

## TASK 2: CONFIGURE THE WORDPRESS MASTER INSTANCE TO USE THE RDS DATABASE

* Rien à signaler.

## TASK 3: CREATE A CUSTOM VIRTUAL MACHINE IMAGE

* Rien à signaler.

## TASK 4: CREATE A LOAD BALANCER

* Rien à signaler.

## TASK 5: LAUNCH A SECOND INSTANCE FROM THE CUSTOM IMAGE

* Dans notre setup, un client ne devrait pas avoir accès directement à une instance EC2 ou au RDS via internet (bien que techniquement possible puisqu'il n'y a pas de restriction d'IP et que tout est publique pour le labo). On veut forcer le passage via le load balancer pour répartir la charge. Il manque également un security group et un pare-feu devant le load balancer.
* Concernant le calcul des coûts, il fallait faire les calculs à la main, en vous basant ceux que vous avez faits aux points précédents.

## TASK 5B: DELETE AND RE-CREATE THE LOAD BALANCER USING THE COMMAND LINE INTERFACE

* Attention à votre export PDF. Parfois certains éléments sont tronqués, comme c'est le cas des commandes CLI dans votre rapport. Le début semble correct, mais je ne peux pas voir la fin.

## TASK 6: TEST THE DISTRIBUTED APPLICATION

* Il manque le report de vegeta (output dans la console) pour montrer les statistiques de la montée en charge.
* Rien à voir avec le labo, mais vous verrez que des personnes mal intentionnées essayent d'accéder à votre serveur web pour y lancer des commandes (premier log du second screenshot). C'est pas de votre faute, les IP sont régulièrement scannées.