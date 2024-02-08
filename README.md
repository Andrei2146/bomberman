Importera sprites 
Gör en tile palette, sätt spriterna där och gör kartan
gör child object indestructible och destructible
sätt tilemap collider till båda gameobjects

Gör player game object
gör child objects, up, down, left, right för animations

Ny script, movement controller
Assign rigidbodyn 

public class movementcontroller : MonoBehaviour
{

   public new Rigidbody2D rigidbody {  get; private set; }



Gör directions för playern

private Vector2 direction = Vector2.down;
public float speed = 5f;

gör input för directions

public KeyCode inputUp = KeyCode.W;
public KeyCode inputDown = KeyCode.S;
public KeyCode inputLeft = KeyCode.A;
public KeyCode inputRight = KeyCode.D;


gör sprite animations
ny script, animation sprite renderer
sätt reference till sprite renderer 

private SpriteRenderer spriteRenderer;

gör att sprite renderer kan bli abled och disabled när man går

private void OnEnable()
{
    spriteRenderer.enabled = true;
}
private void OnDisable()
{
    spriteRenderer.enabled = false;
}


gör en function som tar hand om animation, gör en annam function
gör en variable
public float animationTime = 0.25f;

Varje 1/4 av en sekund animationen kommer att gå till next frame

private void Start()
{
    InvokeRepeating(nameof(NextFrame), animationTime, animationTime);
}


gör en variable som trackar vad frame man är på
private int animationFrame;

gör variable för arrayn för sprites

 public Sprite idleSprite;
 public Sprite[] animationSprites;

sätt variables som gör att animations loopar
public bool loop = true;
kollar om animation är idle
public bool idle = true;


När man byter frame kollar om man måste gå tillbaka till starten, om animationFrame är större än
längden av animationen (hur många sprites man har) börjar animationen på nytt
private void NextFrame()
{
     animationFrame++;

    if (loop && animationFrame >= animationSprites.Length) { 
    animationFrame = 0;
    }
     

    kollar om man är idle, och spelar idle spriten
    om man är inte idle, sprite renderer spriten kommer att bli animation sprite
    if (idle)
    {
        spriteRenderer.sprite = idleSprite;
    } else if (animationFrame >= 0 && animationFrame < animationSprites.Length)
    {
        spriteRenderer.sprite = animationSprites[animationFrame];
    }
}

sätt animation scripten till child objects i player parent
sätt idle spriten och sen sätt hur många frames animationen kommer att ha
sätt spriten till varje frame

gå i movement controller och sätt en reference till sprite animations script
 public AnimationSpriteRenderer spriteRendererUp;
 public AnimationSpriteRenderer spriteRendererDown;
 public AnimationSpriteRenderer spriteRendererLeft;
 public AnimationSpriteRenderer spriteRendererRight;
 public AnimationSpriteRenderer spriteRendererDeath;
kollar vilken sprite renderer är active
 private AnimationSpriteRenderer activeSpriteRenderer;

update direction function, och renderar spriten baserat på direction

private void SetDirection (Vector2 newDirection, AnimationSpriteRenderer spriteRenderer)
{
    direction = newDirection;

    spriteRendererUp.enabled = spriteRenderer == spriteRendererUp;
    spriteRendererDown.enabled = spriteRenderer == spriteRendererDown;
    spriteRendererLeft.enabled = spriteRenderer == spriteRendererLeft;
    spriteRendererRight.enabled = spriteRenderer == spriteRendererRight;


    activeSpriteRenderer = spriteRenderer;
    activeSpriteRenderer.idle = direction == Vector2.zero;
}

gör bomben
ny game object, sätt sprite till den
gör den en prefab
sätt animated sprite renderer till den så att den har animations

bomb controller script
gör reference till bomb prefab
public GameObject bombPrefab;
sätt en key som gör att bomben spawnar
public KeyCode inputKey = KeyCode.LeftShift;

sätt variabler för hur långt det tar för bomben att explodera
 public float bombFuseTime = 3f;

sen hur många bomber man kan ha
 public int bombAmount = 1;
visar hur många bomber man har
private int bombsRemaining;

om bomberna som man har kvar är mera än 1, sätt en bomb
private void Update()
{
    if (bombsRemaining > 0 && Input.GetKeyDown(inputKey)) {
        StartCoroutine(PlaceBomb());
    }
}

Sätter bomben på marken
private IEnumerator PlaceBomb()
{
    Vector2 position = transform.position;
    position.x = Mathf.Round(position.x);
    position.y = Mathf.Round(position.y);

   Gör bomb prefaben appear, och gör att man har mindre bomber kvar
    GameObject bomb = Instantiate(bombPrefab, position, Quaternion.identity);
    bombsRemaining--;
    
    Gör så att man röra bomben
    yield return new WaitForSeconds(bombFuseTime);

    position = bomb .transform.position;
    position.x = Mathf.Round(position.x);
    position.y = Mathf.Round(position.y);
    
    Då bomben exploderar, gör en explosion
    Explosion explosion = Instantiate(explosionPrefab, position, Quaternion.identity);
    explosion.SetActiveRenderer(explosion.start);
    explosion.DestroyAfter(explosionDuration);
    
    Explode(position, Vector2.up, explosionRadius);
    Explode(position, Vector2.down, explosionRadius);
    Explode(position, Vector2.left, explosionRadius);
    Explode(position, Vector2.right, explosionRadius);

    
    Destroy(bomb);
    bombsRemaining++;

gör att man kan inte gå över bomben
private void OnTriggerExit2D(Collider2D other)
{
    if (other.gameObject.layer == LayerMask.NameToLayer("Bomb"))
    {
        other.isTrigger = false;
    }
}

}

explosions game object
sätt animation scripten till den
explosion script
sätt reference till animation script
Animations för explosion
public class Explosion : MonoBehaviour
{
    public AnimationSpriteRenderer start;
    public AnimationSpriteRenderer middle;
    public AnimationSpriteRenderer end;


gör att animations spelar
public void SetActiveRenderer(AnimationSpriteRenderer renderer)
{
    start.enabled = renderer == start;
    middle.enabled = renderer == middle;
    end.enabled = renderer == end;
}

public void SetDirection(Vector2 direction)
{
    float angle = Mathf.Atan2(direction.y, direction.x);
    transform.rotation = Quaternion.AngleAxis(angle * Mathf.Rad2Deg, Vector3.forward);
}

settar directionen av explosion
public void SetDirection(Vector2 direction)
{
    float angle = Mathf.Atan2(direction.y, direction.x);
    transform.rotation = Quaternion.AngleAxis(angle * Mathf.Rad2Deg, Vector3.forward);
}

destroyar explosion
public void DestroyAfter(float seconds)
{
    Destroy(gameObject, seconds);
}

gör reference i bomb script till bomb explosion 
public Explosion explosionPrefab;

sätt en float som tar hand om hur mycket range explosionen har och hur mycket den kommer att vara 
public float explosionDuration = 1f;
public int explosionRadius = 1;

defina vilken direction explosionen kommer att gå

private void Explode(Vector2 position, Vector2 direction, int length)
{
    if(length <= 0) {
        return;
    }

Gör så att explosionen går inte ut ur kartan
    position += direction;

    if (Physics2D.OverlapBox(position, Vector2.one / 2f, 0f, explosionLayerMask))
    {
        ClearDestructible(position);
        return;
    }

    Explosion explosion = Instantiate(explosionPrefab, position, Quaternion.identity);
    explosion.SetActiveRenderer(length > 1 ? explosion.middle : explosion.end);
    explosion.SetDirection(direction);
    explosion.DestroyAfter(explosionDuration);
    

    Explode(position, direction, length - 1);
}

gör destructibles
ny game object destructible
gö den prefab
sätt animation script till den
sätt reference i bomb controller till bomb prefab
public Tilemap destructibleTiles;
public Destructibles DestructiblePrefab;

destructible script

public float destructionTime = 1f;

sätt en ny function i bomb script

Gör att tilen blir destroyed, tilen på kartan blir destroyed och replaced av tile game object och blir animerade
 private void ClearDestructible(Vector2 position)
 {
     Vector3Int cell = destructibleTiles.WorldToCell(position);
     TileBase tile = destructibleTiles.GetTile(cell);

     if (tile != null)
     {
         Instantiate(DestructiblePrefab, position, Quaternion.identity);
         destructibleTiles.SetTile(cell, null);
     }
 }

sätt reference i bomb scripten till destructible tile och dens prefab

gör items som man kan ta
gör prefabs för varje item
item pickup script

Typen av items som man kan få
 public enum ItemType
 {
     ExtraBomb,
     BlastRadius,
     SpeedIncrease,
 }

public ItemType type;

sätter effekt för varje item, och destroyar den då man tar den
private void OnItemPickUp (GameObject player)
{
    switch (type)
    {
        case ItemType.ExtraBomb:
            player.GetComponent<BombController>().AddBomb();
            break;

            case ItemType.BlastRadius:
            player.GetComponent<BombController>().explosionRadius++;
            break;

            case ItemType.SpeedIncrease:
            player.GetComponent<movementcontroller>().speed++;
            break;
    }

    Destroy(gameObject);
}

gör att man kan ta itemen

 private void OnTriggerEnter2D(Collider2D other)
 {
     if (other.CompareTag("Player"))
     {
         OnItemPickUp(other.gameObject);
     }
 }


gör att man har mera bombs då man får bomb item
 public void AddBomb()
 {
     bombAmount++;
     bombsRemaining++;
 }

gör variables för chances av en item att spawna och hur många spawnar för varje tile
public float itemSpawnChance = 0.2f;
public GameObject[] spawnableItems;


gör så att då destructible tile game objekten blir destroyed spawnar man en item
private void OnDestroy()
{
    if(spawnableItems.Length > 0 && Random.value < itemSpawnChance)
    {
        int randomIndex = Random.Range(0, spawnableItems.Length);
        Instantiate(spawnableItems[randomIndex], transform.position, Quaternion.identity);  
    }
}
