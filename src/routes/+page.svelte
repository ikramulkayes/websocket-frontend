<script>
    import { onMount, onDestroy } from 'svelte';
    // import io from 'socket.io-client';

    const userId = Math.floor(Math.random() * 1000000000);
    // @ts-ignore
    let socket;
    // @ts-ignore
    let messages = [];
    // @ts-ignore
    let currentState = "Waiting...";
    let numberOfInstances = 0;
    let isInstancesRunning = false;
    let isNewUserAdding = false;
    let isPostCheckingOn = false;

    // Fields for the POST request
    let orderId = '';
    let profileLink = '';
    let orderAmount = '';
    let maxPerPost = '';

    // onMount(() => {
    //     socket = io('localhost:5000');

    //     socket.on('connect', () => {
    //         console.log('Connected to the server!');
    //         // @ts-ignore
    //         socket.emit('userId', { message: "I am connected now", userId: userId });
    //     });

    //     socket.on('user event', (data) => {
    //         // @ts-ignore
    //         messages = [...messages, data];
    //         console.log(data);
    //     });

    //     socket.on('disconnect', () => {
    //         console.log('Disconnected from the server!');
    //     });
    // });

    // onDestroy(() => {
    //     // @ts-ignore
    //     if (socket) {
    //         socket.disconnect();
    //     }
    // });
    onMount(() => {
        startPolling();
    });

    const sendMessage = () => {
        // @ts-ignore
        // socket.emit('user event', { "message": "I pressed a button", userId: userId });
        console.log("socket")
    }

    const startNewPostsChecker = async () => {
        try {
            const response = await fetch(import.meta.env.VITE_URL+'/start_new_posts_checker', {
                method: 'GET',
                headers: {
                    'Authorization': 'Bearer drdre'
                },
            });

            if (response.ok) {
                console.log('New posts checker started successfully.');
                
                currentState = 'New posts checker started successfully!';
            } else {
                console.error('Failed to start new posts checker.');
            }
        } catch (error) {
            console.error('Error:', error);
        }
    }


    const checkForState = async () => {
        try {
            const response = await fetch(import.meta.env.VITE_URL+'/checkForState', {
                method: 'GET',
                headers: {
                    'Authorization': 'Bearer drdre'
                },
            });
            if (response.ok) {
                const data = await response.json();
                // Update variables based on the response
                numberOfInstances = data.number_of_instances;
                isInstancesRunning = data.is_instances_on;
                isNewUserAdding = data.is_new_user_adding_on;
                isPostCheckingOn = data.is_post_checking_on;
            } else {
                console.error('Failed to fetch state.');
            }
        } catch (error) {
            console.error('Error:', error);
        }
    };

    const startPolling = () => {
        checkForState(); // Initial call
        setInterval(checkForState, 30000); // Poll every 30 seconds
    };







    const endCurrentProcess = async () => {
        try {
            const response = await fetch(import.meta.env.VITE_URL+'/end', {
                method: 'GET',
                headers: {
                    'Authorization': 'Bearer drdre'
                },
            });

            if (response.ok) {
                console.log('process Ended.');
                
                currentState = 'Current Process ended successfully!';
            } else {
                console.error('Failed to End the process.');
            }
        } catch (error) {
            console.error('Error:', error);
        }
    }

    const submitPostRequest = async () => {
        try {
            const response = await fetch(import.meta.env.VITE_URL+'/userid_insert', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer drdre'
                },
                body: JSON.stringify({
                    order_id: orderId,
                    profile_link: profileLink,
                    order_amount: parseInt(orderAmount, 10),
                    max_per_post: parseInt(maxPerPost, 10),
                    comment: true,
                }),
            });

            if (response.ok) {
                console.log('POST request successful.');
                currentState = "User added successfully!"
                currentState = `User added ${profileLink} successfully!`
            } else {
                console.error('Failed to make POST request.');
                currentState = "Failed to add user!"
            }
        } catch (error) {
            console.error('Error:', error);
        }
    }

    // New form fields
    let instances = '';

    const startInstances = async () => {
        try {
            currentState = `Starting ${instances} instances...`
            const response = await fetch(import.meta.env.VITE_URL+`/start?instances=${instances}`, {
                method: 'GET',
                headers: {
                    'Authorization': 'Bearer drdre'
                },
            });

            if (response.ok) {
                console.log(`Started ${instances} instances successfully.`);
                currentState = `Started ${instances} instances successfully!`
            } else {
                console.error(`Failed to start ${instances} instances.`);
                currentState = `Failed to start ${instances} instances!`
            }
        } catch (error) {
            console.error('Error:', error);
        }
    }
</script>

<style>
    @import './css/styles.css';
</style>
<div  style="max-height: 90vh; , overflowY: auto">
<div class="w-full flex flex-row justify-center my-8">
    <div class="flex flex-col justify-center">
        <p class="text-2xl font-bold self-center">Welcome to Instagram Scrapper</p>
        
    </div>
</div>

<div class="w-full h-full flex flex-row justify-center my-8">
    <div class="flex flex-col justify-center">
        <p class="text-2xl font-bold self-center">Current State</p>
        <p class="text-2xl font-bold self-center" style="color: red;">{currentState}</p>
    </div>
</div>

<div class="w-full h-full flex flex-row justify-center my-8">
    {#if isInstancesRunning}
        <div class="flex flex-col justify-center ">
            <p class="text-2xl font-bold self-center">{numberOfInstances} instances Running!</p>
        </div>
    {:else if isNewUserAdding}
        <div class="flex flex-col justify-center ">
            <p class="text-2xl font-bold self-center">New User is being added!</p>
        </div>
    {:else if isPostCheckingOn}
        <div class="flex flex-col justify-center ">
            <p class="text-2xl font-bold self-center">Post checker is on!</p>
        </div>
    {:else}
        <div class="flex flex-col justify-center ">
            <p class="text-2xl font-bold self-center">No process is Running!</p>
        </div>
    {/if}
</div>



<div class="w-full h-full flex flex-row justify-center my-8">
    <button class="bg-blue-500 text-white border border-black p-2 rounded-lg my-4" on:click={startNewPostsChecker}>Check Users Posts</button>
</div>

<div class="w-full h-full flex flex-col justify-center my-8 form-container">
    <h1 class="text-2xl font-bold self-center my-2">Add New Premium User</h1>
    <label for="orderId">Order ID:</label>
    <input type="text" id="orderId" bind:value={orderId} class="form-group" />

    <label for="profileLink">Profile Link:</label>
    <input type="text" id="profileLink" bind:value={profileLink} class="form-group" />

    <label for="orderAmount">Order Amount:</label>
    <input type="text" id="orderAmount" bind:value={orderAmount} class="form-group" />

    <label for="maxPerPost">Max Per Post:</label>
    <input type="text" id="maxPerPost" bind:value={maxPerPost} class="form-group" />

    <button on:click={submitPostRequest}>Add New User</button>
</div>

<!-- New form components -->
<div class="w-full h-full flex flex-col justify-center my-8 new-form-container">
    <h1 class="text-2xl font-bold self-center my-2">Run Scrapper</h1>
    <label for="instances">Number of Instances:</label>
    <input type="text" id="instances" bind:value={instances} class="form-group" />

    <button on:click={startInstances}>Start Instances</button>
</div>
<div class="w-full h-full flex flex-row justify-center my-8">
    <button style="background-color: red;" class="bg-blue-500 text-white border border-black p-2 rounded-lg my-4" on:click={endCurrentProcess}>End Scrapping</button>
</div>
</div>