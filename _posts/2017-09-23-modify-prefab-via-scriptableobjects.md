---
title: Modify Prefab via ScriptableObjects
date: 2017-09-23T20:12:58+00:00
tags:
  - Unity
---
Today I spent a good amount of time on something that won't improve game play. But it gave me a good feeling because I love the moment when you find a neat solution for a problem that's been bugging you.

The problem:

I have multiple hero classes with different health / speed / etc but they all share a lot of components like NavMeshAgents. Same same but different. I want to manage that common ground they share as well as their individuality. I wanted:

  * To edit everything in the unity inspector. I didn't want to open my IDE to make (minor) changes
  * To be able to make changes that affect every hero
  * To be able to make changes that affect only a single hero
  * No duplicate code or settings! Never write anything twice and never have to change something in 2 locations
  * A prefab that I can instantiate

So basically I want slight variations of a generic HeroPrefab that are nice to manage.

Here's how I did it:

## Step 1: Create Generic Hero Prefab 
![heroPrefab](/assets/2017/09/heroPrefab.png)

Here you can see why I didn't want to have a separate prefab for every hero. If I want to change the waypoint prefabs I don't want to do it for every single hero!

## Step 2: Create a CharacterBuilder ScriptableObject

I've watched a couple of videos about how ScriptableObject are basically the next messiah and now finally I found a use for them!
  
If you're not sure what ScriptableObjects are - me neither :P. But here I use them as scripts that can receive input through the inspector like components but without needing a game object.

So what's going on? The MakeHero method on line 12 is the public interface of this class - that's what someone else will be calling with their hero that they want to be modified. Here we set hitspoints, speed and whatever else you want. The values are from the public properties that we will later fill from the inspector.

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
    hero.GetComponent<Damageable>().MaxHitpoints = Hitpoints;
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
      var attack = hero.AddComponent<MeleeAttack>();
      attack.AttackDamage = AttackDamage;
    } else {
      var attack = hero.AddComponent<RangeAttack>();
      attack.ProjectilePrefab = ProjectilePrefab;
    }
  }
}
```

What's this "ModifiyHero" method that doesn't do anything? That's for when you need to write a lot of code for one specific hero and don't want to cluster up the generic class. You'd extend the CharacterBuilder and override this method to add even more customization:

```csharp
public class RogueBuilder : CharacterBuilder {
  public override GameObject ModifyHero(GameObject hero) {
    // Very specific code for rogue heroes here
    return hero;
  }
}
```

Then i used [this editor script](http://www.richardlord.net/blog/unity/creating-scriptableobjects-in-unity.html) to create an asset from my RogueBuilder (which extends the CharacterBuilder). When you select the asset you can edit it, like you would a component:
  
![RogueBuilder](/assets/2017/09/RogueBuilder.png)

If you copy paste my code you will notice that there's no field for "AttackDamage" or "ProjectilePrefab" - that's because they are both marked to be hidden (line 08 & 10). We only want one of them to be visible depending on if Melee or Range is selected. For this we need a CustomEditor.

## Step 3: CustomEditor (optional)

Having both "AttackDamage" and "ProjectilePrefab" visible even though only 1 will be used isn't a problem, but it isn't neat either. So I decided to fix it.
  
On line 04 we call the base method - if we didn't do this we would have to manually create a field for every single property! Too much work! So we build the default inspector (which hides "AttackDamage" & "ProjectilePrefab") and then add only on of those depending on the selection of AttackType (line 9 & 11-12).

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

Now I drag the generic hero prefab and the RogueBuilder asset in the editor into my RaidManager (that's who decideds what heroes we get):
  
![RaidManager](/assets/2017/09/RaidManager.png)

```csharp
public class RaidManager : MonoBehaviour {
  public GameObject HeroPrefab;
  public List<CharacterBuilder> Builders;

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

Note: The code is a simplified version of what I actually use - that's why the screenshot & the code don't match 100%.

**And there you have it!** You have a generic HeroPrefab where your changes affect all heroes and you have a HeroBuilder Asset for every hero where you can override whatever you want!

One more thing: As you can see in the last code we modify the hero after it's been instantiated. This means the Awake() lifecycle hook has already been called. In my case this caused characters to not start with full health since the current health was set to the maximum health in the Awake() function. And our CharacterBuilder increased the maximum health after that. My solution was to write a setter that increases the current hitpoints by the same amount. This is especially handy since there might be a spell ingame that increases max hp and it would be nice if the current hp would increase as well.

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
