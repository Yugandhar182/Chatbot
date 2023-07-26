<script>
	import { onMount, onDestroy } from 'svelte';
	import { initializeApp } from 'firebase/app';
	import { getDatabase, ref, push, onValue, remove } from 'firebase/database';
	import 'bootstrap/dist/css/bootstrap.min.css';
	import UserGreeting from './UserGreeting.svelte'; 
  
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
	let hours = date.getHours();
	const minutes = date.getMinutes();
	const ampm = hours >= 12 ? 'pm' : 'am';
	hours %= 12;
	hours = hours || 12;
	const formattedTime = `${hours}:${minutes.toString().padStart(2, '0')} ${ampm}`;
	return formattedTime;
  }
  
	// Function to add a new online user
	function addOnlineUser() {
	  if (fullName.trim() !== '') {
		push(onlineUsersRef, { fullName }).then(() => {
		  joinedChat = true; // Set the flag to indicate user joined the chat
		});
	  }
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
   
  
	// Function to clear all messages from the chat
	function clearData() {
	  // Remove all messages
	  const messagesRef = ref(db, 'messages');
	  remove(messagesRef);
	}
  
	function showUserMessages(selectedUserName) {
	  fullName = selectedUserName;
	  // Filter the messages based on the selected user's name
	  messages = messages.filter(
		(messageData) => messageData.fullName === fullName
	  );
	}
	
	let messageContainer; 
  
	function scrollToBottom() {
	  if (messageContainer) {
		messageContainer.scrollTop = messageContainer.scrollHeight;
	  }
	}
  
	onMount(scrollToBottom);
  
  
  
  
  </script>
	
	  <!--<button on:click={clearData}>Clear Data</button> -->
	 
	  <main>
		<div class="main-container">
		  {#if !joinedChat}
		  
			<h1 style="color: blue;">Enter your name to join the Chat</h1>
			<div>
			  <input type="text" id="fullName" bind:value={fullName} placeholder="Enter your name" />
			  <button on:click={addOnlineUser}>Join Chat</button>
			</div>
		  {:else}
  
		  <UserGreeting userDisplayName={fullName} />
  
			<div class="chat-container" on:scroll={scrollToBottom}>
			  <div class="message-container" bind:this={messageContainer}>
				{#each messages as messageData}
				  <div class="chat-message {messageData.fullName === fullName ? 'sender' : 'receiver'}">
					<span class="chat-sender">{messageData.fullName}:</span>
					<span class="chat-content">{messageData.message}</span>
					<span class="chat-timestamp">{formatDate(messageData.timestamp)}</span>
				  </div>
				{/each}
			  </div>
			  <div class="input-container">
				<input type="text" bind:value={message} />
				<div class="button-container">
				<button style="background-color: blue;" on:click={sendMessage}>Send</button>
			  </div>
			  </div>
			</div>
			<div class="online-users-container">
			  <div class="Headingcard-body">
				<h2>Online Users</h2>
			  </div>
			  <div class="online-users-list">
				{#each onlineUsers as user}
				  <div class="card-body" >
					<p class="username">{user.fullName}</p>
				  </div>
				{/each}
			  </div>
			</div>
		   
		  {/if}
	   
		</div>
	  </main>
	  
  
  <style>
  
  .main-container{
	  margin-left: 470px;
	  margin-top: -40px;
   
	
	}
	.online-users-container {
	  border: 1px solid #ccc;
	  padding: 20px;
	  width: 300px;
	  height: 580px;
	  margin-top:-600px ;
	  margin-left: -300px;
	  overflow: auto; 
	  margin-right: 100px;
		 background-color: rgb(241, 249, 249);
	
	}
   
  .online-users-list {
	  position: relative;
	  margin-top: 10px;
	  overflow-y: auto; 
	}
  
   
   
   .input-container {
	  position: absolute;
	  bottom: 15px; /* Adjust the value as needed for the desired distance from the bottom */
	  left: 10px; /* Adjust the value as needed for the desired distance from the left */
	  display: flex;
	  align-items: center;
	  width: 800px;
	  margin-left:500px;
	  }
  
	.input-container input {
	  margin-right: 5px;
	  width: 700px;
	  height: 30px;
	  margin-bottom:1px;
  
	}
	.button-container{
	
	  margin-bottom:1px;
  
	}
	.message-container {
	 background-color: rgb(241, 249, 249);
	   border-radius: 5px;
	   margin-top: -700px;
		margin-right:100px;
		height: 590px;
		border: 1px solid #ccc;
	  width:850px;
	  margin: 10px;
	  justify-content: center;
	  padding: 20px;
	  overflow: auto;
	  }
  .chat-message {
	  margin: 5px;
	  border: 1px solid #ccc;
	  border-radius: 5px;
	  padding: 10px;
	  display: flex;
	  align-items: center;
	  width: 700px;
	  margin-left:80px;
	  height:30px;
	 
	}
  
	.chat-sender {
	  font-weight: bold;
	  margin-right: 10px;
	
	}
  
	.chat-timestamp {
	  margin-left: auto;
	}
  
	.chat-message.sender {
	 
	  color: black; 
	}
  
	.chat-message.receiver {
	 
	  color: blue;
	}
	.card-body {
	  padding: 10px;
	  border: 1px solid #ccc;
	  border-radius: 5px;
	  margin-bottom: 5px;
	  background-color: #f9f9f9;
	  height: 40px;
	}
	.Headingcard-body {
	  padding: 10px;
	  border: 1px solid #ccc;
	  border-radius: 5px;
	  margin-bottom: 5px;
	  background-color: #f9f9f9;
	  height: 60px;
	}
	.username {
	
	  margin-right: 60px; 
	}
	button {
	  /* Add styles for the remove button */
	  background-color: red; /* Customize the button background color */
	  color: white; /* Customize the button text color */
	  border: none;
	  padding: 5px 10px;
	  border-radius: 5px;
	  cursor: pointer;
	  margin-left: 10px; /* Adjust the margin-left value as needed */
	}
  
  
  </style>