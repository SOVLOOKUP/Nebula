<script>
	import { url } from '@roxi/routify';
</script>

<div class="login">
  <center
    ><img src="favicon.svg" alt="logo" />
    <p style="color:#000000">Welcome to Nebula Graph Analysis</p></center
  >
  <a href={$url('/analysis')}>
    <button>Login</button>
  </a>
  
</div>

<style>
  .login {
    position: absolute;
    width: 399px;
    height: 428px;
    left: calc(50% - 399px / 2 + 0.5px);
    top: calc(50% - 428px / 2);

    background: rgba(155, 184, 221, 0.568);
    box-shadow: inset -20px 24px 31px -5px rgba(6, 7, 9, 0.25);
    border-radius: 25px;
  }
  .login button {
    position: absolute;
    width: 223px;
    height: 65px;
    left: calc(50% - 223px / 2 + 0.5px);
    top: calc(50% - 65px / 2 + 97.5px);

    /* background: linear-gradient(90deg, #2400FF 0%, rgba(89, 22, 175, 0.296875) 33.25%, rgba(255, 255, 255, 0) 100%), #0500FF; */
    border-radius: 14px;
  }
</style>
