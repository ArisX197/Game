using UnityEngine;
using UnityEngine.SceneManagement;

public class PlayerController : MonoBehaviour
{
    [Header("Movement")]
    public float moveSpeed = 5f;
    public float jumpForce = 10f;
    private Rigidbody2D rb;
    private bool isGrounded = true;

    [Header("Effects")]
    public ParticleSystem deathParticles;
    public ParticleSystem jumpParticles;
    public AudioClip jumpSound;
    public AudioClip deathSound;
    private AudioSource audioSource;

    [Header("Skin")]
    public Color playerColor = Color.white;
    private SpriteRenderer spriteRenderer;

    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
        audioSource = GetComponent<AudioSource>();
        spriteRenderer = GetComponent<SpriteRenderer>();
        
        // Load player color
        LoadPlayerColor();
    }

    void Update()
    {
        // 1. Auto-move forward
        transform.Translate(Vector2.right * moveSpeed * Time.deltaTime);

        // 2. Handle Jump Input (PC & Mobile)
        if (Input.GetKeyDown(KeyCode.Space) || Input.GetMouseButtonDown(0))
        {
            if (isGrounded)
            {
                Jump();
            }
        }

        // Rotate cube based on velocity (optional, but looks good)
        float rotation = rb.velocity.y * 2f;
        transform.rotation = Quaternion.Euler(0, 0, rotation);
    }

    void Jump()
    {
        rb.velocity = new Vector2(rb.velocity.x, 0); // Reset vertical velocity
        rb.AddForce(Vector2.up * jumpForce, ForceMode2D.Impulse);
        isGrounded = false;
        
        // Effects
        audioSource.PlayOneShot(jumpSound);
        if (jumpParticles)
            jumpParticles.Play();
    }

    void OnCollisionEnter2D(Collision2D collision)
    {
        // Check if we hit the ground
        if (collision.gameObject.CompareTag("Ground"))
        {
            isGrounded = true;
        }

        // Check if we hit an obstacle
        if (collision.gameObject.CompareTag("Obstacle"))
        {
            Die();
        }
    }
    
    // Also check for triggers (e.g., airborne spikes)
    void OnTriggerEnter2D(Collider2D other)
    {
        if (other.gameObject.CompareTag("Obstacle"))
        {
            Die();
        }
    }

    void Die()
    {
        // Play effects
        audioSource.PlayOneShot(deathSound);
        if (deathParticles)
        {
            // Move particles to player position and play
            deathParticles.transform.position = transform.position;
            deathParticles.Play();
        }

        // Notify GameManager to restart
        GameManager.instance.HandleDeath();

        // Hide player
        gameObject.SetActive(false);
    }
    
    void LoadPlayerColor()
    {
        // Load from PlayerPrefs, default to white
        float r = PlayerPrefs.GetFloat("PlayerColorR", 1f);
        float g = PlayerPrefs.GetFloat("PlayerColorG", 1f);
        float b = PlayerPrefs.GetFloat("PlayerColorB", 1f);
        playerColor = new Color(r, g, b);
        spriteRenderer.color = playerColor;
    }
}
