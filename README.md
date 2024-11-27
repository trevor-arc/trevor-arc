

public class PlayerController : MonoBehaviour
{
    public float moveSpeed = 5f;
    public float jumpHeight = 5f;
    public GameObject weapon;  // Reference to the weapon object
    public Transform cameraTransform;

    private Rigidbody rb;
    private bool isGrounded;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    void Update()
    {
        MovePlayer();
        Jump();
        Shoot();
    }

    void MovePlayer()
    {
        float moveX = Input.GetAxis("Horizontal");
        float moveZ = Input.GetAxis("Vertical");

        Vector3 moveDirection = cameraTransform.forward * moveZ + cameraTransform.right * moveX;
        moveDirection.y = 0;

        rb.MovePosition(transform.position + moveDirection * moveSpeed * Time.deltaTime);
    }

    void Jump()
    {
        isGrounded = Physics.Raycast(transform.position, Vector3.down, 1.1f);
        if (isGrounded && Input.GetButtonDown("Jump"))
        {
            rb.AddForce(Vector3.up * jumpHeight, ForceMode.VelocityChange);
        }
    }

    void Shoot()
    {
        if (Input.GetMouseButtonDown(0))  // Left mouse button to shoot
        {
            Ray ray = new Ray(cameraTransform.position, cameraTransform.forward);
            RaycastHit hit;

            if (Physics.Raycast(ray, out hit))
            {
                if (hit.collider.CompareTag("Enemy"))
                {
                    // Deal damage to enemy
                    Debug.Log("Hit: " + hit.collider.name);
                }
            }
        }
    }
}
using Mirror;

public class PlayerNetwork : NetworkBehaviour
{
    private PlayerController playerController;

    void Start()
    {
        if (isLocalPlayer) // Only the local player will have control
        {
            playerController = GetComponent<PlayerController>();
        }
    }

    void Update()
    {
        if (isLocalPlayer)
        {
            // Handle player input here, e.g., movement and shooting
            playerController.MovePlayer();
            playerController.Jump();
            playerController.Shoot();
        }
    }
}


public class FlyingBus : MonoBehaviour
{
    public Transform[] dropZones;
    public float speed = 10f;

    void Update()
    {
        transform.Translate(Vector3.forward * speed * Time.deltaTime);

        if (Input.GetKeyDown(KeyCode.J)) // Simulate jump
        {
            DropPlayer();
        }
    }

    void DropPlayer()
    {
        // Randomly select a drop zone
        Transform dropZone = dropZones[Random.Range(0, dropZones.Length)];
        GameObject player = FindObjectOfType<PlayerController>().gameObject;
        player.transform.position = dropZone.position;
    }
}


public class ItemSpawner : MonoBehaviour
{
    public GameObject[] items;  // Array of item prefabs (e.g., weapons, health)
    public Vector3 spawnAreaMin;
    public Vector3 spawnAreaMax;

    void Start()
    {
        SpawnItems();
    }

    void SpawnItems()
    {
        foreach (GameObject item in items)
        {
            Vector3 randomPosition = new Vector3(
                Random.Range(spawnAreaMin.x, spawnAreaMax.x),
                1f,  // Set an appropriate height for items to spawn
                Random.Range(spawnAreaMin.z, spawnAreaMax.z)
            );
            Instantiate(item, randomPosition, Quaternion.identity);
        }
    }
}


public class GameManager : MonoBehaviour
{
    public Text victoryText;
    public GameObject[] players;

    void Update()
    {
        players = GameObject.FindGameObjectsWithTag("Player");
        if (players.Length == 1) // If only one player is left
        {
            victoryText.text = "#1 Victory Royale!";
        }
    }
}
