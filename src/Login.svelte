<script>
  // import { signup, login } from './user';
  import { user } from './user';

  let username = $state('');
  let password = $state('');

  function login() {
    user.auth(username, password, ({err}) => err && alert(err));
  }

  function handleSignup() {
    user.create(username, password, ({err}) => {
      if(err) {
        alert(err);
      }else {
        login();
      }
    });
  }
  
  // const handleSignup = async () => {
  //   try {
  //     await signup(username, password);
  //     login(username, password);
  //   }
  //   catch (error) {
  //     console.error(error);
  //   }
  // };

</script>

<label for="username">Username</label>
<input type="text" name="username" bind:value={username} minlength="3" maxlength="16" />

<label for="password">Password</label>
<input type="password" name="password" bind:value={password} minlength="6" maxlength="32" />

<button class="login" onclick={login}>Log In</button>
<button class="login" onclick={handleSignup}>Sign Up</button>