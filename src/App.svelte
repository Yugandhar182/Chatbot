<script>
	import { onMount, onDestroy } from 'svelte';
	import { initializeApp } from 'firebase/app';
	import { getDatabase, ref, push, onValue, remove } from 'firebase/database';
	import 'bootstrap/dist/css/bootstrap.min.css';
   
  
	// Firebase configuration (replace with your own values)
	const firebaseConfig = {
	  apiKey: 'AIzaSyAYnBrsz0_EgcCs5QuAUptdOBC4s1GHffg',
   
	  databaseURL: 'https://chatbot-6dc5f-default-rtdb.firebaseio.com/',
	  projectId: 'chatbot-6dc5f',
	
	};
  
	// Initialize Firebase
	const firebaseApp = initializeApp(firebaseConfig);
	
  
	let fullName = '';
	let message = '';
	let messages = [];
	let onlineUsers = [];
	let joinedChat = false;
	let showModal = false;
  
	// Firebase Realtime Database references
	const db = getDatabase(firebaseApp);
	const messagesRef = ref(db, 'messages');
	const onlineUsersRef = ref(db, 'onlineUsers');
	// Function to send a new message
	function sendMessage() {
	  if (message.trim() !== '') {
		push(messagesRef, {
		  fullName,
		  message,
		  timestamp: {
			'.sv': 'timestamp', // ServerValue.TIMESTAMP equivalent in v9
		  },
		});
		message = '';
	  }
	}
  
	function formatDate(timestamp) {
	  const date = new Date(timestamp);
	  return `${date.getHours()}:${date.getMinutes()}`;
	}
  
	// Function to add a new online user
	function addOnlineUser() {
	  if (fullName.trim() !== '') {
		push(onlineUsersRef, { fullName }).then(() => {
		  joinedChat = true; // Set the flag to indicate user joined the chat
		});
	  }
	}
  
	// Function to remove the user from the onlineUsersRef when the component is destroyed
	function removeOnlineUser() {
	  remove(onlineUsersRef.child(fullName)).then(() => {
		joinedChat = false; // Set the flag to indicate user left the chat
	  });
	}
  
	// Listen for new messages and online users
	onMount(() => {
	  onValue(messagesRef, (snapshot) => {
		messages = Object.values(snapshot.val() || {});
	  });
  
	  onValue(onlineUsersRef, (snapshot) => {
		onlineUsers = Object.values(snapshot.val() || {});
		console.log("Online Users:", onlineUsers);
	  });
	});
  
	// Call removeOnlineUser when the component is destroyed (user leaves the chat)
	onDestroy(removeOnlineUser);
  
	// Function to clear all messages from the chat
	function clearData() {
	  // Remove all messages
	  const messagesRef = ref(db, 'messages');
	  remove(messagesRef);
	}
  
  </script>
  <main>
   
	<div class="main-container">
	{#if !joinedChat}
	<h1 style="color:blue;" > Enter the name to join Chat</h1>
	<div>
	 
	  <input type="text" id="fullName" bind:value={fullName} placeholder="Enter your name" />
	  <button on:click={addOnlineUser}>Join Chat</button>
	</div>
	{:else}
		<div>
		  <h2>Welcome, {fullName}!</h2>
		  <div class="message-container">
			{#each messages as messageData}
			  <div
				class="chat-message {messageData.fullName === fullName ? 'sender' : 'receiver'}">
				<span class="chat-sender">{messageData.fullName}:</span>
				<span class="chat-content">{messageData.message}</span>
				<span class="chat-timestamp">{formatDate(messageData.timestamp)}</span>
			  </div>
			{/each}
			<div class="input-container">
			  <input type="text" bind:value={message} />
			  <button style="background-color:blue;" on:click={sendMessage}>Send</button>
			</div>
		  </div>
		  <button on:click={clearData}>Clear Data</button>
		</div>
	  {/if}
	</div>
  </main>
  
  
  <style>
   
   .input-container {
	  position: absolute;
	  bottom: 10px; /* Adjust the value as needed for the desired distance from the bottom */
	  left: 10px; /* Adjust the value as needed for the desired distance from the left */
	  display: flex;
	  align-items: center;
	  
	}
  
	.input-container input {
	  margin-right: 5px;
	  width: 1300px;
  
	}
	.message-container {
	  background-image: url('');
	  border: 1px solid #ccc;
	  border-radius: 5px;
	  padding: 10px;
	  margin-left: -80px;
	  width: 1400px;
	  height: 600px;
	  /* Add the following styles to position the input container at the bottom */
	  position: relative;
	}
  
  .chat-message {
	  margin: 5px;
	  border: 1px solid #ccc;
	  border-radius: 5px;
	  padding: 10px;
	  display: flex;
	  align-items: center;
	}
  
	.chat-sender {
	  font-weight: bold;
	  margin-right: 10px;
	}
  
	.chat-timestamp {
	  margin-left: auto;
	}
  
	.chat-message.sender {
	  /* Style for sender messages (optional) */
	  color: black; /* You can customize this color */
	}
  
	.chat-message.receiver {
	  /* Style for receiver messages (blue color) */
	  color: blue;
	}
  
  </style>
  