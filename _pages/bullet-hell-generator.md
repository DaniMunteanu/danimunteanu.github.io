---
permalink: /projects/bullet-hell-generator
layout: single
author_profile: true
author: Daniel Munteanu
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Bullet-Hell Generator

## Short Description

In this **3D Unity** mini-project, the user can generate projectile shooting patterns similar to those used in Bullet-Hell games.

There are multiple parameters the player can modify, including:
- Number of projectiles per bullet ring
- Fire rate
- Rotation angle
- Bullet acceleration

Other interesting features, explained in this page:
- Projectile Object Pooling
- Spherical patterns
- Waving patterns
- Helical patterns

This project can also be found on 
<i class="fa-brands fa-github"></i>
<a href="https://github.com/DaniMunteanu/SM_BulletHell" target="_blank" rel="noopener noreferrer">Github</a> 
and 
<i class="fa-brands fa-itch-io"></i>
<a href="https://danimunteanu.itch.io/bullet-hell-generator" target="_blank" rel="noopener noreferrer">itch.io</a> 

## Gameplay Video

{% include video id="MqPcpqAztvQ" provider="youtube" %}

## Playable Web Build

<div class="itchio-widget-wrapper">
    <iframe frameborder="0" src="https://itch.io/embed/5111058?bg_color=141010&amp;fg_color=ffffff&amp;link_color=f21368&amp;border_color=5d5d5d" width="552" height="167"><a href="https://danimunteanu.itch.io/bullet-hell-generator">Bullet-Hell Generator by DaniMunteanu, fishy-business, straw-berizz</a></iframe>
</div>

## Implemented Mechanics

### Projectile Object Pooling

Simply creating and destroying every bullet inside the scene will eventually take up a considerable amount of CPU power, if the number and frequency of spawned projectiles is high enough. To improve performance, I used the **Object Pooling** design patern. Instead of instancing a new bullet, each fired projectile is retrieved from a list of already instanced, but inactive, bullets. When a bullet is retrieved from the pool, it becomes active inside the scene. When the retrieved bullet needs to dissapear, it becomes inactive, going back in the pool to be used as another bullet in the future.

{% include figure popup=true image_path="/assets/images/bullet-hell-generator/object-pool.png" alt="Object pool inside the Unity editor" caption="Object pool inside the Unity editor" %}

```csharp
public class ObjectPool : MonoBehaviour
{
    public static ObjectPool SharedInstance;
    private List<GameObject> pooledObjects;
    public GameObject objectToPool;
    public int amountToPool;

    void Awake()
    {
        SharedInstance = this;
    }

    void Start()
    {
        // This list will hold all instantiated bullets
        pooledObjects = new List<GameObject>();
        GameObject tmp;
        // Instantiate the bullets and set them to inactive
        for(int i = 0; i < amountToPool; i++)
        {
            tmp = Instantiate(objectToPool);
            tmp.SetActive(false);
            pooledObjects.Add(tmp);
        }
    }

    // Returns an inactive bullet from the pool, or null if all bullets are active
    public GameObject GetPooledObject()
    {
        for(int i = 0; i < amountToPool; i++)
        {
            if(!pooledObjects[i].activeInHierarchy)
            {
                return pooledObjects[i];
            }
        }
        return null;
    }
}
```

The ObjectPool is accessed inside the script that handles shooting all the bullets.

```csharp
public class PatternGenerator : MonoBehaviour
{
    public void SpawnBullet(int indexInRing, float horizontalAngleSpacing, float horizontalWiggleModifier, float verticalWiggleModifier)
    {
        //...
        Bullet newBullet = ObjectPool.SharedInstance.GetPooledObject()?.GetComponent<Bullet>();
        if (newBullet == null)
        {
            return;
        }
        else
        {
            newBullet.gameObject.SetActive(true);
        }
        //...
    }
}
```

### Spherical patterns

{% include figure popup=true image_path="/assets/images/bullet-hell-generator/spherical-preview.gif" alt="Spherical pattern" caption="Spherical pattern based on how many bullets a ring has"%}

To determine the position of a bullet in 3D space, I used these formulas:

{% include figure popup=true image_path="/assets/images/bullet-hell-generator/equations/eq1.png"%}
{% include figure popup=true image_path="/assets/images/bullet-hell-generator/equations/eq2.png"%}
{% include figure popup=true image_path="/assets/images/bullet-hell-generator/equations/eq3.png"%}

where 

β = vertical angle (up/down tilt towards the horizontal plane XZ)

θ = horizontal angle (rotation around the vertical axis Y)

r = distance from the origin point (0,0,0)

{% include figure popup=true image_path="/assets/images/bullet-hell-generator/spherical-pattern.png" alt="Spherical ring distribution" caption="Spherical ring distribution"%}

If we want to create a spherical shape, we need to distribute the bullet rings using this formula:

{% include figure popup=true image_path="/assets/images/bullet-hell-generator/equations/eq4.png"%}

where

j = bullet ring index

M = number of even spaces between rings

```csharp
// simplified version of the code
public class PatternGenerator : MonoBehaviour
{
    public void FireBullets()
    {
        startPoint = transform.position;
        float horizontalAngleSpacing = 360f / numberOfBullets;
        float horizontalWiggleModifier = horizontalWiggleSize * Mathf.Sin(wiggleSpeed * Time.time);
        float verticalWiggleModifier = verticalWiggleSize * Mathf.Cos(wiggleSpeed * Time.time);
        
        // Check if the bullets should form a sphere
        if (sphereMode)
        {
            for (verticalLayerIndex = 0; verticalLayerIndex <= numberOfSphereParts; verticalLayerIndex++)
            {
                verticalAngle = Mathf.Asin(1 - ((2 * verticalLayerIndex)/numberOfSphereParts));
                
                for (int i = 0; i < numberOfBullets; i++)
                {
                    SpawnBullet(i, horizontalAngleSpacing, horizontalWiggleModifier, verticalWiggleModifier);
                }
            }
        }
        // If not in sphere mode, spawn bullets in a ring pattern
        else
        {
            verticalAngle = 0;
            for (int i = 0; i < numberOfBullets; i++)
            {
                SpawnBullet(i, horizontalAngleSpacing, horizontalWiggleModifier, verticalWiggleModifier);
            }
        }
    }

    public void SpawnBullet(int indexInRing, float horizontalAngleSpacing, float horizontalWiggleModifier, float verticalWiggleModifier)
    {
        // Considering the bullets shouldn't rotate or wiggle
        float bulletDirXPosition = startPoint.x + Mathf.Cos(((horizontalAngle) * Mathf.PI) / 180f) * Mathf.Cos(verticalAngle) * radius;
        float bulletDirYPosition = startPoint.y + Mathf.Sin(verticalAngle) * radius;
        float bulletDirZPosition = startPoint.z + Mathf.Sin(((horizontalAngle) * Mathf.PI) / 180f) * Mathf.Cos(verticalAngle) * radius;

        Vector3 newPositionVector = new Vector3(bulletDirXPosition, bulletDirYPosition, bulletDirZPosition);
        Vector3 bulletDirection = (newPositionVector - startPoint).normalized;

        // A new bullet is created...
        
        newBullet.SetMoveDirection(bulletDirection);
    }
}
```

### Waving patterns

{% include figure popup=true image_path="/assets/images/bullet-hell-generator/waving-preview.gif" alt="Horizontal and vertical wiggle" caption="Rotating spherical pattern, with horizontal and vertical wiggle"%}

For the bullets to have a waving effect, the horizontal and vertical angles should change according to these formulas:

{% include figure popup=true image_path="/assets/images/bullet-hell-generator/equations/eq5.png"%}
{% include figure popup=true image_path="/assets/images/bullet-hell-generator/equations/eq6.png"%}

where

w = wiggle speed

A_β = vertical wiggle intensity

A_θ = horizontal wiggle intensity

```csharp
float horizontalWiggleModifier = horizontalWiggleSize * Mathf.Sin(wiggleSpeed * Time.time);
float verticalWiggleModifier = verticalWiggleSize * Mathf.Cos(wiggleSpeed * Time.time);
for (int i = 0; i < numberOfBullets; i++)
{
    SpawnBullet(i, horizontalAngleSpacing, horizontalWiggleModifier, verticalWiggleModifier);
}

// Same spawn method, now using wiggle parameters
public void SpawnBullet(int indexInRing, float horizontalAngleSpacing, float horizontalWiggleModifier, float verticalWiggleModifier)
{
    float bulletDirXPosition = startPoint.x + Mathf.Cos(((horizontalAngle + horizontalWiggleModifier) * Mathf.PI) / 180f) * Mathf.Cos(verticalAngle) * radius;
    float bulletDirYPosition = startPoint.y + Mathf.Sin(verticalAngle + verticalWiggleModifier) * radius;
    float bulletDirZPosition = startPoint.z + Mathf.Sin(((horizontalAngle + horizontalWiggleModifier) * Mathf.PI) / 180f) * Mathf.Cos(verticalAngle) * radius;
}
```

### Helical patterns

{% include figure popup=true image_path="/assets/images/bullet-hell-generator/helical-preview.gif" alt="Helical bullet pattern"%}

To create a helical effect, we need to adjust the vertical angle based on how many times we've fired:

{% include figure popup=true image_path="/assets/images/bullet-hell-generator/equations/eq7.png"%}

where

φ = speed in pattern evolution (phi)

K = how many times we've fired

```csharp
public class PatternGenerator : MonoBehaviour
{
    public void FireBullets()
    {
        // ...
        
        // Check if the bullets should form a sphere
        if (sphereMode)
        {
            for (verticalLayerIndex = 0; verticalLayerIndex <= numberOfSphereParts; verticalLayerIndex++)
            {
                verticalAngle = Mathf.Asin(1 - ((2 * verticalLayerIndex)/numberOfSphereParts)) + Mathf.PI / 180f * Mathf.Sin(phi * currentShot);
                
                for (int i = 0; i < numberOfBullets; i++)
                {
                    SpawnBullet(i, horizontalAngleSpacing, horizontalWiggleModifier, verticalWiggleModifier);
                }
            }
        }
        // If not in sphere mode, spawn bullets in a ring pattern
        else
        {
            //...
        }
        currentShot += 1;
    }
}
```
