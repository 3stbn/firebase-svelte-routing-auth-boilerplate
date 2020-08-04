<script>
    import { user } from '../stores'
    import { push } from 'svelte-spa-router'
    import { getContext } from 'svelte'
    const { auth } = getContext('firebase').getFirebase()
    let email
    let password
    let errorMessage

    $: if (email || password) {
        errorMessage = null
    }

    async function login() {
        try {
            await auth().signInWithEmailAndPassword(email, password)
            push('/dashboard')
        } catch (error) {
            if (error.message) {
                email = ''
                password = ''
                errorMessage = error.message
            }
        }
    }
</script>

<div class="container">
    <div class="section">
        <h1 class="title">Login</h1>
        <hr />
        {#if errorMessage}
            <p class="help is-danger">{errorMessage}</p>
        {/if}
        <form on:submit|preventDefault={login}>
            <div class="field">
                <label class="label">Email</label>
                <div class="control">
                    <input type="text" bind:value={email} class="input" required class:is-danger={errorMessage} />

                </div>
            </div>
            <div class="field">
                <label class="label">Password</label>
                <div class="control">
                    <input
                        type="password"
                        bind:value={password}
                        class="input"
                        required
                        class:is-danger={errorMessage} />
                </div>
            </div>
            <div class="control">
                <input type="submit" class="button is-info is-light" value="Submit" />
            </div>
        </form>
        <hr />
        <p>
            Do not have an account ?
            <a href="#/signup">Sign Up</a>
        </p>
        <hr />
    </div>

</div>
