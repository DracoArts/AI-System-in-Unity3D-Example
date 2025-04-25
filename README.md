
# Welcome to DracoArts

![Logo](https://dracoarts-logo.s3.eu-north-1.amazonaws.com/DracoArts.png)



# AI System in Unity 
This AI system provides a robust framework for creating intelligent NPCs (Non-Player Characters) in Unity, featuring pathfinding, decision-making, sensory detection, and state-based behaviors. The system is built using Unity's NavMesh for navigation and a Finite State Machine (FSM) approach for managing different AI behaviors.


## 1. Core AI Components

### A. AI State Machine (FSM)

The AI operates using a state-driven system, transitioning between different behaviors based on conditions like player proximity, noise detection, and health status.

### States:

### 1. Idle

- The AI stands still, observing its surroundings.

- Transitions to Patrol after a delay or to Chase if it detects the player.

### 2.Patrol

- The AI follows a predefined path (waypoints).

- Waits briefly at each patrol point before moving to the next.

- If the player is detected, it switches to Chase mode.

### 3. Chase

- The AI pursues the player using NavMesh pathfinding.

- If the player gets too far, it returns to Patrol or Idle.

- If the player is within attack range, it switches to Attack.

### 4 .Attack

- The AI stops moving and faces the player.

- Triggers attack animations or damage logic.

- If the player moves out of range, it resumes Chase.

### B. AI Senses (Detection System)

- The AI detects the player using vision and hearing:

#### Vision Detection:
##### Field of View (Cone-shaped vision)

- The AI can only see the player if they are within a defined angle (e.g., 60°).

- Range-based detection (e.g., 10 meters).

- Obstacle checks (walls block vision).

#### Hearing Detection:
- The AI can detect the player based on sound radius.

- Loud actions (gunshots, footsteps) increase detection chance.

- The AI may investigate noise sources even if it doesn't see the player.

## C. Navigation & Movement

- #### Uses Unity’s NavMeshAgent for smooth pathfinding.

- #### Adjustable speeds for different states (Patrol Speed vs. Chase Speed).

-  ####   Obstacle avoidance (dynamic rerouting if blocked).


## D. Optional Advanced Features


#### AI Health System

- Tracks HP and triggers death/respawn logic.

- Can modify behavior (e.g., flee when health is low).

#### Animation Controller

- Syncs animations with AI states (walking, attacking, dying).

- Uses Animator parameters (e.g., IsMoving, IsAttacking).

### Behavior Extensions

- Investigate State: AI checks suspicious noises.

- Flee State: AI runs away when at low health.

- Group Coordination: Multiple AI units communicate (e.g., flanking).

## 2. How It Works in Practice
## Initialization

- The AI starts in Idle or Patrol mode.

- It continuously checks for the player using vision & hearing.

## Detection & Reaction

- If the player is seen/heard, the AI switches to Chase.

- If the player hides, the AI may return to Patrol after losing track.

- If the player is in attack range, the AI stops moving and attacks.

## Dynamic Adjustments

- The AI recalculates paths if obstacles appear.

- It can be customized with different aggression levels (e.g., cautious vs. relentless).

# 3. Ideal Use Cases
- Enemies in FPS/TPS games (zombies, soldiers, monsters).

- NPCs in open-world games (guards, animals, civilians).

- Stealth game opponents (reacting to noise and vision).

## Setup Instructions
###  Required Components:

- Add a NavMeshAgent component to your AI character

- Bake a NavMesh in your scene (Window > AI > Navigation)

### AI Character Setup:

- Attach both AIStateMachine and AISenses scripts to your AI character

- Set up patrol points by creating empty GameObjects and assigning them to the patrolPoints array

- Configure detection ranges, speeds, and other parameters in the inspector

### Player Setup:

- Make sure your player has the "Player" tag

- If you want hearing detection, implement a noise system on your player

#### State Machine Configuration:

- The AI will automatically transition between states based on distance to player

- You can modify the transition logic in the Update method

## Conclusion
- This AI system provides a modular, efficient, and scalable solution for Unity games, 
  supporting basic patrol/chase mechanics while allowing advanced expansions like squad
  behavior or machine learning. By adjusting parameters (vision range, speed, detection rules), you can create anything from dumb zombies to highly tactical enemies.



## Usage/Examples

 #### AI Controller

    using UnityEngine;
    using UnityEngine.AI;

    public enum AIState { Idle, Patrol, Chase, Attack }

    public class AIStateMachine : MonoBehaviour
    {
    [Header("AI Settings")]
    public AIState currentState = AIState.Patrol;
    public float detectionRange = 10f;
    public float attackRange = 2f;
    public float patrolSpeed = 2f;
    public float chaseSpeed = 4f;
    public float rotationSpeed = 5f;
    
    [Header("Patrol Settings")]
    public Transform[] patrolPoints;
    public float waitTimeAtPoint = 2f;
    
    private NavMeshAgent agent;
    private Transform player;
    private float stateTimeElapsed;
    private int currentPatrolIndex;
    private bool isWaiting;
    
    private void Awake()
    {
        agent = GetComponent<NavMeshAgent>();
        player = GameObject.FindGameObjectWithTag("Player").transform;
    }
    
    private void Update()
    {
        float distanceToPlayer = Vector3.Distance(transform.position, player.position);
        
        // State transitions
        switch(currentState)
        {
            case AIState.Idle:
                if(distanceToPlayer <= detectionRange)
                    ChangeState(AIState.Chase);
                else if(ShouldPatrol())
                    ChangeState(AIState.Patrol);
                break;
                
            case AIState.Patrol:
                if(distanceToPlayer <= detectionRange)
                    ChangeState(AIState.Chase);
                break;
                
            case AIState.Chase:
                if(distanceToPlayer <= attackRange)
                    ChangeState(AIState.Attack);
                else if(distanceToPlayer > detectionRange * 1.5f)
                    ChangeState(AIState.Patrol);
                break;
                
            case AIState.Attack:
                if(distanceToPlayer > attackRange)
                    ChangeState(AIState.Chase);
                break;
        }
        
        // State behavior
        switch(currentState)
        {
            case AIState.Idle:
                IdleBehavior();
                break;
                
            case AIState.Patrol:
                PatrolBehavior();
                break;
                
            case AIState.Chase:
                ChaseBehavior();
                break;
                
            case AIState.Attack:
                AttackBehavior();
                break;
        }
    }
    
    private void ChangeState(AIState newState)
    {
        currentState = newState;
        stateTimeElapsed = 0f;
        
        switch(newState)
        {
            case AIState.Idle:
                agent.isStopped = true;
                break;
                
            case AIState.Patrol:
                agent.speed = patrolSpeed;
                agent.isStopped = false;
                SetNextPatrolPoint();
                break;
                
            case AIState.Chase:
                agent.speed = chaseSpeed;
                agent.isStopped = false;
                break;
                
            case AIState.Attack:
                agent.isStopped = true;
                break;
        }
    }
    
    private void IdleBehavior()
    {
        // Just wait in place
        stateTimeElapsed += Time.deltaTime;
    }
    
    private void PatrolBehavior()
    {
        if(agent.remainingDistance <= agent.stoppingDistance && !agent.pathPending)
        {
            if(!isWaiting)
            {
                isWaiting = true;
                stateTimeElapsed = 0f;
            }
            
            if(stateTimeElapsed >= waitTimeAtPoint)
            {
                isWaiting = false;
                SetNextPatrolPoint();
            }
        }
        
        stateTimeElapsed += Time.deltaTime;
    }
    
    private void ChaseBehavior()
    {
        agent.SetDestination(player.position);
    }
    
    private void AttackBehavior()
    {
        // Face the player
        Vector3 direction = (player.position - transform.position).normalized;
        Quaternion lookRotation = Quaternion.LookRotation(new Vector3(direction.x, 0, direction.z));
        transform.rotation = Quaternion.Slerp(transform.rotation, lookRotation, Time.deltaTime * rotationSpeed);
        
        // Attack logic would go here
        Debug.Log("Attacking player!");
    }
    
    private void SetNextPatrolPoint()
    {
        if(patrolPoints.Length == 0) return;
        
        currentPatrolIndex = (currentPatrolIndex + 1) % patrolPoints.Length;
        agent.SetDestination(patrolPoints[currentPatrolIndex].position);
    }
    
    private bool ShouldPatrol()
    {
        return patrolPoints.Length > 0 && stateTimeElapsed > 3f; // Wait 3 seconds before patrolling
    }
    
    
     }




### AIAnimationController

     using UnityEngine;
    [RequireComponent(typeof(Animator), typeof(AIStateMachine))]
    public class AIAnimationController : MonoBehaviour
    {
    private Animator animator;
    private AIStateMachine stateMachine;
    private NavMeshAgent agent;
    
    private void Awake()
    {
        animator = GetComponent<Animator>();
        stateMachine = GetComponent<AIStateMachine>();
        agent = GetComponent<NavMeshAgent>();
    }
    
    private void Update()
    {
        // Update animator parameters based on AI state
        animator.SetFloat("Speed", agent.velocity.magnitude);
        
        switch(stateMachine.currentState)
        {
            case AIState.Idle:
                animator.SetBool("IsMoving", false);
                animator.SetBool("IsAttacking", false);
                break;
                
            case AIState.Patrol:
                animator.SetBool("IsMoving", true);
                animator.SetBool("IsAttacking", false);
                break;
                
            case AIState.Chase:
                animator.SetBool("IsMoving", true);
                animator.SetBool("IsAttacking", false);
                break;
                
            case AIState.Attack:
                animator.SetBool("IsMoving", false);
                animator.SetBool("IsAttacking", true);
                break;
        }
    }
    
    // Animation event for attack impact
    public void AttackImpact()
    {
        // Check if player is in range and deal damage
        // You would need to implement this based on your game's    combat    system
    }
    
     }
## Images

 Ai 
![](https://github.com/AzharKhemta/Gif-File-images/blob/main/Ai%20system%20unity.gif?raw=true)


## Authors

- [@MirHamzaHasan](https://github.com/MirHamzaHasan)
- [@WebSite](https://mirhamzahasan.com)


## 🔗 Links

[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/company/mir-hamza-hasan/posts/?feedView=all/)
## Documentation

[Unity Ai Documentation](https://docs.unity3d.com/2021.3/Documentation/Manual/com.unity.modules.ai.html)




## Tech Stack
**Client:** Unity,  AI ,C#




