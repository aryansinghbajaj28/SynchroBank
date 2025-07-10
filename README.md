# SynchroBank
Overview
This is a robust multi-threaded client-server banking system built in C. It allows multiple clients to concurrently perform banking operations such as account creation, login, deposit, withdrawal, and transfer. The system ensures thread safety, data consistency, and high concurrency through POSIX threads, mutex locks, and semaphores.

System Workflow
1. Server Startup (server.c)
Initializes:

Per-account mutexes (pthread_mutex_t customerMutex[i])

Global bank mutex (pthread_mutex_t bankMutex)

Semaphore for periodic reporting (sem_t actionLock)

Spawns two key threads:

Session Acceptor Thread (session_acceptor_thread()):

Accepts new client connections

Spawns client_session_thread() for each client

Periodic Action Thread (periodic_action_cycle_thread()):

Triggers every 20 seconds using SIGALRM and setitimer()

Reports number of active connections and logged-in users

2. Client Startup (client.c)
Connects to the server (sock_to_server())

Displays main menu (bank_menu())

3. Banking Operations
a. Account Creation
Client sends: "create <account_name>"

Server:

Checks for duplicate using search_account()

Allocates a free slot in customers[]

Stores account name

b. Login
Client sends: "serve <account_name>"

Server:

Verifies account existence and if it's already in use

Marks account as in-use

c. After Login: Access Account
Client accesses a sub-menu via access_account() with:

query: View account balance

deposit <amount>

withdraw <amount>

transfer <target_account>\n<amount>

end: Logout

d. Server-side Execution
Each command is parsed and dispatched to:

deposit: Adds funds (with mutex lock)

withdraw: Validates and deducts (with mutex lock)

transfer: Locks both source and target accounts in consistent order

query: Reads data under mutex protection

end: Resets account usage status

Optimizations & Concurrency Handling
 Multi-threading
Each client handled in a separate pthread

Prevents blocking and allows simultaneous sessions

 Mutex Locks
Per-account mutexes:

Protect balance and status flags during deposit, withdrawal, and transfer

Bank-wide mutex:

Used during login, account creation, and periodic reporting

Ensures thread-safe access to shared resources

 Semaphores
Used to coordinate the periodic status printing thread (actionLock)

Avoids CPU busy-waiting using sem_wait()

 Deadlock Prevention
Transfer operations acquire locks in a fixed order (source first, then target)

Mutexes are held for minimal duration only around critical sections

 Periodic Monitoring
Every 20 seconds, the server logs:

Number of active connections

Existing accounts

Accounts currently in use

Implemented via:

SIGALRM signal

setitimer() interval timer

sem_post() to trigger action in monitoring thread
