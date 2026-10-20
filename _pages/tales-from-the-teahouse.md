---
permalink: /projects/tales-from-the-teahouse
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "cog"
layout: single
author_profile: true
author: Daniel Munteanu
---

# Tales from the Teahouse

## Short Description

This game is a **2D Unity** project developed in collaboration with 
<a href="https://www.jamk.fi/en" target="_blank" rel="noopener noreferrer">Jamk University of Applied Sciences</a> through the <a href="https://eudres.eu/" target="_blank" rel="noopener noreferrer">EUDRES program</a>
as a micro-internship experience.

Our team was responsible for developing a teaser game promoting the Jamk University’s video-game therapy
classes. The game is a visual novel focused on conversations with customers, with each character showcasing
different topics addressed by the mindfulness program.

My tasks included implementing some of the game’s core features:
- Drag-and-drop ingredient system
- Tooltips
- Audio manager
- Ingredients tracking and validation systems
- Tutorial
- The cup’s particle system
- UI responsiveness

This project can also be found on 
<a href="https://github.com/DaniMunteanu/EudresInternship" target="_blank" rel="noopener noreferrer">Github</a> 
and 
<a href="https://danimunteanu.itch.io/tales-from-the-teahouse" target="_blank" rel="noopener noreferrer">itch.io</a> 

## Gameplay Video

{% include video id="o2s75bFVd6w" provider="youtube" %}

## Playable Web Build

<div class="itchio-widget-wrapper">
    <iframe frameborder="0" src="https://itch.io/embed/4857904?bg_color=141010&amp;fg_color=ffffff&amp;link_color=f21368&amp;border_color=5d5d5d" width="552" height="167"><a href="https://danimunteanu.itch.io/tales-from-the-teahouse">Tales From the Teahouse by DaniMunteanu, Bogdaain, 1Fearzz6, ruediuqqq, straw-berizz</a></iframe>
</div>

## Implemented Features

For smooth property transitions (such as objects floating up & down, or ingredients tilting), I used the
<a href="https://assetstore.unity.com/packages/tools/animation/dotween-hotween-v2-27676?srsltid=AU7gw4WbVe4ZEiEUYkE9oNRvV0iTxFwNZrUdbL50xA3_bV3zhEyFDKU1" target="_blank" rel="noopener noreferrer">DOTween asset</a> 
from the **Unity Asset Store**.

### Draggable Ingredients

In order to easily add/remove ingredients to our game, I stored their data inside Scriptable Objects. In the context of our game, each ingredient corresponds to their own list of _keywords_. These are also stored inside Scriptable Objects. The player's objective is to add all the ingriedients matching a list of required keywords.

{% include figure popup=true image_path="/assets/images/tales-from-the-teahouse/ingredient-so.png" alt="Assigning Keywords to an Ingredient" caption="Assigning Keywords to an Ingredient" %}

{% include figure popup=true image_path="/assets/images/tales-from-the-teahouse/draggable-ingredients-preview.gif" alt="Dragging and dropping an ingredient, with visual feedback" caption="Dragging and dropping an ingredient, with visual feedback" %}

```csharp
public class DraggableIngredient : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    [SerializeField] public Ingredient ingredientData;
    //other variables...

    //Start() method ...
    //OnBeginDrag() method ...

    public void OnDrag(PointerEventData eventData)
    {
        if (!isDraggable)
            return;

        RectTransform parentRect = rectTransform.parent as RectTransform;

        // Update the position of the ingredient to follow the mouse cursor
        if (RectTransformUtility.ScreenPointToLocalPointInRectangle(
            parentRect,
            eventData.position,
            eventData.pressEventCamera,
            out Vector2 localPoint))
        {
            rectTransform.anchoredPosition = localPoint;
        }

        //Check if the ingredient is over the cup, which implements the IDropArea interface
        ingredientCollider.enabled = false;
        Collider2D hitCollider = Physics2D.OverlapPoint(rectTransform.position);
        ingredientCollider.enabled = true;

        if (hitCollider != null && hitCollider.TryGetComponent<IDropArea>(out IDropArea dropArea))
        {
            // Rotate the ingredient clockwise by 45 degrees when over the cup
            rectTransform.DORotate(new Vector3(0, 0, 45f), 0.2f)
                .SetEase(Ease.InOutSine)
                .SetLink(gameObject)
                .SetUpdate(true);
        }
        else
        {
            // Rotate the ingredient back to its original rotation when not over the cup
            rectTransform.DORotate(Vector3.zero, 0.2f)
                .SetEase(Ease.InOutSine)
                .SetLink(gameObject)
                .SetUpdate(true);
        }
    }

    public void OnEndDrag(PointerEventData eventData)
    {
        if (!isDraggable)
            return;

        //Check if the ingredient is over the cup, which implements the IDropArea interface
        ingredientCollider.enabled = false;
        Collider2D hitCollider = Physics2D.OverlapPoint(rectTransform.position);
        ingredientCollider.enabled = true;

        if (hitCollider != null && hitCollider.TryGetComponent<IDropArea>(out IDropArea dropArea))
        {
            dropArea.OnIngredientDropped(ingredientData);
        }

        rectTransform.DOKill();

        rectTransform.anchoredPosition = initialPosition;
        rectTransform.rotation = initialRotation;
    }
}
```

### Ingredient and Keyword Tracking

There are 3 ingredient types, and there is a limit of how many ingredients of the same type the player can add.

{% include figure popup=true image_path="/assets/images/tales-from-the-teahouse/ingredient-type-so.png" alt="Assigning Keywords to an Ingredient" caption="Assigning Keywords to an Ingredient" %}

{% include figure popup=true image_path="/assets/images/tales-from-the-teahouse/particles-preview.gif" alt="Visual feedback upon adding ingredients" caption="Visual feedback upon adding ingredients" %}

If the ingredient added is of type _base_, the cup should "fill" up with a sprite. If the added ingredient is a _fruit_ or _addon_, then particles should be emitted from the cup. Below is a simplified version of the cup's script.

```csharp
public class Cup : MonoBehaviour, IDropArea
{
    [SerializeField] IngredientTracker ingredientTracker;

    public void OnIngredientDropped(Ingredient ingredient)
    {
        // If this is the first "Base" ingredient being added, change the cup's filling image to the corresponding sprite
        if (ingredient.ingredientType.typeName == "Base" && fillingImage.sprite == null)
        {
            fillingImage.sprite = ingredient.cupChangeSprite;
            fillingImage.color = Color.white; // Make the image visible
            fillingImage.SetNativeSize(); // Adjust the size of the image to match the sprite

            frontFillingImage.sprite = ingredient.cupChangeSprite;
            frontFillingImage.color = Color.white; // Make the image visible
            frontFillingImage.SetNativeSize(); // Adjust the size of the image to match the sprite
        }
        
        ingredientTracker.AddIngredient(ingredient);
    }
```

{% include figure popup=true image_path="/assets/images/tales-from-the-teahouse/trackers.png" alt="Ingredients are tracked on the blackboard, and remaining keywords are displayed as bubbles" caption="Ingredients are tracked on the blackboard, and remaining keywords are displayed as bubbles" %}

```csharp
public class IngredientTracker : MonoBehaviour
{
    [SerializeField] private KeywordGroup requiredKeywords;

    public void AddIngredient(Ingredient ingredient)
    {
        if (IsIngredientValid(ingredient))
        {
            if (ingredient.name == "Biscuit")
            {
                // Instead of emitting particles, the biscuit will only be shown as a sprite next to the cup
                teaParticles.ShowBiscuit();
            }
            else if (ingredient.ingredientType.typeName != "Base")
            {
                teaParticles.PlayParticles(ingredient.particleSprite);
            }

            ingredientTexts[currentIngredientIndex].text = ingredient.ingredientName;
            currentIngredientIndex++;

            AudioManager.Instance.PlaySFX(AudioManager.Instance.ingredientDrop);

            // Check if the added ingredient has any keywords that are in the requiredKeywords list
            for (int i = 0; i < ingredient.keywords.Length; i++)
            {
                if (requiredKeywords.keywords.Contains(ingredient.keywords[i]))
                {
                    KeywordsTracker.Instance.PopBubble(ingredient.keywords[i]);
                    break;
                }
            }
        }
    }
}
```

{% include figure popup=true image_path="/assets/images/tales-from-the-teahouse/keyword-group-so.png" alt="Required keywords are stored inside a Scriptable Object" caption="Required keywords are stored inside a Scriptable Object" %}

We can use this Scriptable Object to add new keywords from a separate scene, such as the one used during dialogue. Everytime the player asks the NPC a question, a new keyword is added and is stored in the Scriptable Object to be displayed in the tea-preparation scene.

{% include figure popup=true image_path="/assets/images/tales-from-the-teahouse/dialogue-preview.gif" alt="New keyword appears in the form of a bubble" caption="New keyword appears in the form of a bubble" %}

```csharp
public class DialogueManager : MonoBehaviour
{
    // Advances to the next question set or shows the make tea button if all sets are completed
    void AdvanceToNextSet(Keyword keyword = null)
    {
        if (keyword != null)
        {
            KeywordsTracker.Instance.AddBubble(keyword);
            //other lines of code...
        }

        currentSetIndex++;

        if (currentSetIndex < TeaSession.CurrentRecipe.questionSets.Count)
        {
            clickToContinueOverlay.SetActive(true);
        }
        else
        {
            ShowMakeTeaButton();
        }
    }
}
```

```csharp
public class KeywordsTracker : MonoBehaviour
{
    [SerializeField] private KeywordGroup requiredKeywords;

    public void AddBubble(Keyword keyword)
    {
        for (int index = 0; index < bubbles.Length; index++)
        {
            // Add the keyword to the first available bubble slot
            if (bubbles[index].keyword == null)
            {
                requiredKeywords.keywords.Add(keyword);

                bubbles[index].keyword = keyword;
                bubbles[index].gameObject.SetActive(true);
                bubbles[index].Float(); // Start floating animation for the new bubble
                break;
            }
        }
    }
}
```

### Tooltips

{% include figure popup=true image_path="/assets/images/tales-from-the-teahouse/tooltip.png" alt="Tooltip inside the Unity editor" caption="Tooltip inside the Unity editor" %}

```csharp
public class Tooltip : MonoBehaviour
{
    public void SetText(string titleContent, string detailsContent)
    {
        titleTextField.text = titleContent;
        detailsTextField.text = detailsContent;
        RefreshSize();
    }

    public void RefreshSize()
    {
        // Enable the layout element if the details text exceeds the preferred width
        layoutElement.enabled = detailsTextField.preferredWidth >= layoutElement.preferredWidth;
    }
}
```

The tooltip's visibility and position is handled by a **TooltipManager**.

```csharp
public class TooltipManager : MonoBehaviour
{
    public void Show(string text, string details, string[] keywords, Vector3 position = default)
    {
        tooltip.keepPosition = false;
        tooltip.SetText(text, details);
        tooltip.gameObject.SetActive(true);

        tooltip.GetComponent<RectTransform>().position = mainCamera.WorldToScreenPoint(position);
    }
}
```

A **TooltipTrigger** is the area where the player needs to hover/tap for the tooltip to be displayed.  

```csharp
public class TooltipTrigger : MonoBehaviour, ISelectHandler, IDeselectHandler, IPointerClickHandler, IBeginDragHandler, IDragHandler, IEndDragHandler, IPointerEnterHandler, IPointerExitHandler
{
    public void OnSelect(BaseEventData eventData)
    {
        if (isDragging)
            return;
        if (eventData is PointerEventData pointerEventData)
        {
            TooltipManager.Instance.Show(text, details, keywords, rectTransform.position);
        }
    }

    public void OnDeselect(BaseEventData eventData)
    {
        TooltipManager.Instance.Hide();
    }

    public void OnPointerClick(PointerEventData eventData)
    {
        // This ensures that the object can also be selected by touchscreen devices, which don't have a hover state
        eventData.selectedObject = gameObject;
    }

    public void OnPointerEnter(PointerEventData eventData)
    {
        if (eventData is ExtendedPointerEventData extendedPointerEventData)
        {
            // If the pointer event is from a touchscreen, do not show the tooltip
            if (extendedPointerEventData.device is Touchscreen)
            {
                return;
            }
        }

        // Don't show the tooltip while the object is being dragged
        if (isDragging)
            return;
        
        TooltipManager.Instance.Show(text, details, keywords, rectTransform.position);
    }

    public void OnPointerExit(PointerEventData eventData)
    {
        if (eventData is ExtendedPointerEventData extendedPointerEventData)
        {   
            // If the pointer event is from a touchscreen, do not hide the tooltip
            if (extendedPointerEventData.device is Touchscreen)
            {
                return;
            }
        }

        TooltipManager.Instance.Hide();
    }
}
```