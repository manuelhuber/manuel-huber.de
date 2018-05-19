---
id: 20
title: Modify Prefab via ScriptableObjects
date: 2017-09-23T20:12:58+00:00
author: Manuel
layout: post
guid: http://manuel-huber.de/?p=20
permalink: /2017/09/23/modify-prefab-via-scriptableobjects/
image: /wp-content/uploads/2017/09/RogueBuilder.png
categories:
  - Unity
---
Today I spent a good amount of time on something that won&#8217;t improve game play. But it gave me a good feeling because I love the moment when you find a neat solution for a problem that&#8217;s been bugging you.

The problem:

I have multiple hero classes with different health / speed / etc but they all share a lot of components like NavMeshAgents. Same same but different. I want to manage that common ground they share as well as their individuality. I wanted:

  * To edit everything in the unity inspector. I didn&#8217;t want to open my IDE to make (minor) changes
  * To be able to make changes that affect every hero
  * To be able to make changes that affect only a single hero
  * No duplicate code or settings! Never write anything twice and never have to change something in 2 locations
  * A prefab that I can instantiate

So basically I want slight variations of a generic HeroPrefab that are nice to manage.

Here&#8217;s how I did it:

## Step 1: Create Generic Hero Prefab 

<img src="https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/heroPrefab.png?resize=491%2C612" alt="" width="491" height="612" class="alignnone size-full wp-image-21" srcset="https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/heroPrefab.png?w=491 491w, https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/heroPrefab.png?resize=241%2C300 241w, https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/heroPrefab.png?resize=180%2C224 180w, https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/heroPrefab.png?resize=417%2C520 417w" sizes="(max-width: 491px) 100vw, 491px" data-recalc-dims="1" />

Here you can see why I didn&#8217;t want to have a separate prefab for every hero. If I want to change the waypoint prefabs I don&#8217;t want to do it for every single hero!

## Step 2: Create a CharacterBuilder ScriptableObject

I&#8217;ve watched a couple of videos about how ScriptableObject are basically the next messiah and now finally I found a use for them!
  
If you&#8217;re not sure what ScriptableObjects are &#8211; me neither :P. But here I use them as scripts that can receive input through the inspector like components but without needing a game object.

So what&#8217;s going on? The MakeHero method on line 12 is the public interface of this class &#8211; that&#8217;s what someone else will be calling with their hero that they want to be modified. Here we set hitspoints, speed and whatever else you want. The values are from the public properties that we will later fill from the inspector.

```csharp
public class CharacterBuilder : ScriptableObject {
    public int Hitpoints;
    // and other properties you want to modify on most heroes

    // There's "Melee" and "Ranged"
    public AttackType AttackType;
    // Only used for melee
    [HideInInspector] public int AttackDamage;
    // only used for ranged
    [HideInInspector] public GameObject ProjectilePrefab;

    public GameObject MakeHero(GameObject hero) {
        hero.GetComponent&lt;Damageable&gt;().MaxHitpoints = Hitpoints;
        // modify other properties here as well
        AddAttack(hero);
        ModifyHero(hero);
        return hero;
    }

    private virtual GameObject ModifyHero(GameObject hero) {
        return hero;
    }

    private void AddAttack(GameObject hero) {
        if (AttackType == AttackType.Melee) {
            var attack = hero.AddComponent&lt;MeleeAttack&gt;();
            attack.AttackDamage = AttackDamage;
        } else {
            var attack = hero.AddComponent&lt;RangeAttack&gt;();
            attack.ProjectilePrefab = ProjectilePrefab;
        }
    }
}
```

What&#8217;s this &#8220;ModifiyHero&#8221; method that doesn&#8217;t do anything? That&#8217;s for when you need to write a lot of code for one specific hero and don&#8217;t want to cluster up the generic class. You&#8217;d extend the CharacterBuilder and override this method to add even more customization:

```csharp
public class RogueBuilder : CharacterBuilder {
    public override GameObject ModifyHero(GameObject hero) {
        // Very specific code for rogue heroes here
        return hero;
    }
}
```

Then i used [this editor script](http://www.richardlord.net/blog/unity/creating-scriptableobjects-in-unity.html) to create an asset from my RogueBuilder (which extends the CharacterBuilder). When you select the asset you can edit it, like you would a component:
  
<img src="https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/RogueBuilder.png?resize=488%2C203" alt="" width="488" height="203" class="alignnone size-full wp-image-37" srcset="https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/RogueBuilder.png?w=488 488w, https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/RogueBuilder.png?resize=300%2C125 300w, https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/RogueBuilder.png?resize=224%2C93 224w" sizes="(max-width: 488px) 100vw, 488px" data-recalc-dims="1" />

If you copy paste my code you will notice that there&#8217;s no field for &#8220;AttackDamage&#8221; or &#8220;ProjectilePrefab&#8221; &#8211; that&#8217;s because they are both marked to be hidden (line 08 & 10). We only want one of them to be visible depending on if Melee or Range is selected. For this we need a CustomEditor.

## Step 3: CustomEditor (optional)

Having both &#8220;AttackDamage&#8221; and &#8220;ProjectilePrefab&#8221; visible even though only 1 will be used isn&#8217;t a problem, but it isn&#8217;t neat either. So I decided to fix it.
  
On line 04 we call the base method &#8211; if we didn&#8217;t do this we would have to manually create a field for every single property! Too much work! So we build the default inspector (which hides &#8220;AttackDamage&#8221; & &#8220;ProjectilePrefab&#8221;) and then add only on of those depending on the selection of AttackType (line 9 & 11-12).

```csharp
[CustomEditor(typeof(CharacterBuilder), true, isFallback = true)]
public class CharacterBuilderEditor : Editor {
    public override void OnInspectorGUI() {
        base.OnInspectorGUI();
        var builder = target as CharacterBuilder;
        if (builder == null) return;

        if (builder.AttackType == AttackType.Melee) {
            builder.AttackDamage = EditorGUILayout.IntField("Melee Damage", builder.AttackDamage);
        } else {
            builder.ProjectilePrefab = (GameObject) EditorGUILayout.ObjectField("Projectile Prefab",
                builder.ProjectilePrefab, typeof(GameObject), true);
        }
    }
}
```

## Step 4: Building the hero

Now I drag the generic hero prefab and the RogueBuilder asset in the editor into my RaidManager (that&#8217;s who decideds what heroes we get):
  
<img src="https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/RaidManager.png?resize=486%2C195" alt="" width="486" height="195" class="alignnone size-full wp-image-50" srcset="https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/RaidManager.png?w=486 486w, https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/RaidManager.png?resize=300%2C120 300w, https://i2.wp.com/manuel-huber.de/wp-content/uploads/2017/09/RaidManager.png?resize=224%2C90 224w" sizes="(max-width: 486px) 100vw, 486px" data-recalc-dims="1" />

```csharp
public class RaidManager : MonoBehaviour {
    public GameObject HeroPrefab;
    public List&lt;CharacterBuilder&gt; Builders;

    private void Awake() {
        Builders.ForEach(SpawnHero);
    }

    private void SpawnHero(CharacterBuilder builder) {
        var spawn = new Vector3();
        var heroInstance = Instantiate(HeroPrefab, spawn, Quaternion.identity);
        heroInstance = builder.MakeHero(heroInstance);
        // lots of other stuff happening aswell
    }
}
```

Note: The code is a simplified version of what I actually use &#8211; that&#8217;s why the screenshot & the code don&#8217;t match 100%.

**And there you have it!** You have a generic HeroPrefab where your changes affect all heroes and you have a HeroBuilder Asset for every hero where you can override whatever you want!

One more thing: As you can see in the last code we modify the hero after it&#8217;s been instantiated. This means the Awake() lifecycle hook has already been called. In my case this caused characters to not start with full health since the current health was set to the maximum health in the Awake() function. And our CharacterBuilder increased the maximum health after that. My solution was to write a setter that increases the current hitpoints by the same amount. This is especially handy since there might be a spell ingame that increases max hp and it would be nice if the current hp would increase as well.

```csharp
public int MaxHitpoints {
    get { return maxHitpoints; }
    set {
        var dif = value - maxHitpoints;
        maxHitpoints = value;
        hitpoints += dif;
    }
}
```
