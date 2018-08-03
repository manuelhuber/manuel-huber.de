---
title: State of the Game Part II
date: 2017-09-28T20:12:20+00:00
tags:
  - DKP
  - Unity
---
Almost forgot to write this today...

### Health & Attacks

My health system is pretty straight forward. I have a generic _Damagable_ class that adds hitpoints, a healthbar UI above the game object and exposes a public function _ModifyHitpoints_. Negative numbers deal damage, positive numbers heal. Everything that deals damage (spells, projectiles, melee attacks ...) simply call that function with the amount of damage they deal. This way the attacker is in charge of the amount of damage and the _Damagable_ can still modify that before any hitpoints actually get changed. For this the _Damagable_ also exposes a function
```csharp
public void AddDamageInterceptorWithDuration(DamageInterceptor interceptor,
                                             float duration) {
  damageInterceptors.Add(interceptor);
  StartCoroutine(UnityUtil.DoAfterDelay(
    () => RemoveDamageInterceptor(interceptor),
    duration));
}
```
_DamageInterceptors_ are just a function that take an int and return an int - that's the damage modifying function. And an order so you can control the order they are applied in (additions or multiplications first?). 

```csharp
public class DamageInterceptor {
  public int Order = 5;
  public Func<int, int> Interceptor;
}
```

The Damagble then applies all DamageInterceptors before chaning hitpoints (line 6).

```csharp
public void ModifyHitpoints(int initialAmount) {
  var orderedInterceptors = damageInterceptors.OrderBy(interceptor => interceptor.Order);

  var amount =
    orderedInterceptors.Aggregate(initialAmount, (acc, i) => i.Interceptor(acc));
  Hitpoints += amount;
  // Updating healthbars, triggering float combat text, checking for dead etc
}
```

I used this for a "vulnerability" ability where affected characters take more damage. This one I've made as a monobehaviour. So simply _AddComponent<Vulnerability>_ to your target at done! The actual interceptor is on line 18-24

```csharp
public class Vulnerability : MonoBehaviour {
  public int PercentileIncrease = 25;
  public float Duration = 5;

  private void Start() {
    var target = GetComponent<Damageable>();
    if (target != null) {
      ApplyDebuff(target);
    }
    Destroy(this);
  }

  private void ApplyDebuff(Damageable target) {
    var interceptor = new DamageInterceptor {
      // Apply this effect pretty much last
      Order = 10,
      Interceptor = amount => {
        // Don't change healing
        if (amount >= 0) return amount;
        // Increase dmg
        var p = (float) PercentileIncrease / 100;
        return (int) (amount + amount * p);
      }
    };

    target.AddDamageInterceptorWithDuration(interceptor, Duration);
  }
}
```

Other ideas for interceptor functions:

Invulnerability - negate all damage but apply healing. This would need to have a high order so it's applied after after additive dmg modifiers.

```csharp
(amount) => Matf.Max(0,amount);
```

Curse that prevents healing and cause 1 point of damage for every 2 points of attempted heal.

```csharp
(amount) => amount < 0 ? amount*-0.5 : amount;
```

You could even use this to simply check damage without modifying

```csharp
(amount) => {
  StatisticsService.saveData(amount);
  return amount;
}
```

I also spent a good amount of time on the actual attack components aswell as the question "Who knows what?" because characters need to move in order to attack and some movements should be interrupted by attacking, but I didn't want cyclic dependencies... So now the PlayerController orchestrates the MeleeAttack / RangeAttack & WaypointHandler and they don't know about each other. Attack scripts only say if they are in range or not. But this post is already long enough I think. Here's a video of some auto-attacks!

<video autoplay="autoplay" loop="loop" width="768" height="512">
  <source src="/assets/2017/09/autoAttacks.mp4" type="video/mp4">
</video>
