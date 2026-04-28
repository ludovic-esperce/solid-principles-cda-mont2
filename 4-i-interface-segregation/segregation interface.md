<div style="font-family: 'Comic Sans MS', cursive;">

# Ségrégation d’interface :

## Définition 

>**aucun client** ne devrait **dépendre** de **méthodes** qu'il n'**utilise pas**.  
>Il vaut mieux **plusieurs interfaces spécifiques** à une **classe** qu’une **grosse interface générique**.  
>Il ne **faut pas obliger** à **implémenter** des **méthodes** que l’**on ne veut pas**.

Voici une image pour illustrer le propos :  

![Image montrant principe ségrégation d'interface](https://media.licdn.com/dms/image/v2/D5612AQHkPeMhslJ4uA/article-cover_image-shrink_600_2000/article-cover_image-shrink_600_2000/0/1701971208056?e=1778716800&v=beta&t=g9Rs9fJk1wv4DVPaQqRNEDNSGrKgaB4suggwCkZflQU)

Si on fait ceci, la **classe chien** est **obligée d'implémenter** la **méthode** miauler.

```java
public interface Animal {
	void manger();
	void aboyer();
	void miauler();
}

public class Chien implements Animal {
    @Override
    public void manger() {
        //
    }

    @Override
    public void aboyer() {
        //
    }

    @Override
    public void miauler() {
        //
    }
}

public class Chat implements Animal {
    @Override
    public void manger() {
        //
    }

    @Override
    public void aboyer() {
        //
    }

    @Override
    public void miauler() {
        //
    }
}
```
Il vaut mieux faire ceci :

```java
public interface Animal {
    void manger();
}

public interface AnimalQuiAboie {
    void aboyer();
}

public interface AnimalQuiMiaule {
    void miauler();
}

public class Chien implements Animal, AnimalQuiAboie {
    @Override
    public void manger() {
        //
    }

    @Override
    public void aboyer() {
        //
    }
}

public class Chat implements Animal, AnimalQuiMiaule {
    @Override
    public void manger() {
        //
    }

    @Override
    public void miauler() {
        //
    }
}
```
</div>