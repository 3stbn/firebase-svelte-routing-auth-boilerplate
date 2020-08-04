<script>
    import { user, creatingUser } from '../stores'

    import { push } from 'svelte-spa-router'
    import { getContext, tick } from 'svelte'

    const { auth, firestore } = getContext('firebase').getFirebase()

    let email
    let password
    let errorMessage

    $: if (email) {
        errorMessage = null
    }

    async function signup() {
        $creatingUser = true
        try {
            await auth().createUserWithEmailAndPassword(email, password)
            await firestore().collection('users').doc($user.uid).set({ additional: 'test' })
            push('/dashboard')
        } catch (error) {
            if (error.message) {
                email = ''
                password = ''
                errorMessage = error.message
            }
        }
        $creatingUser = false
    }
</script>

<div class="container">
    <div class="section">
        <h1 class="title">Sign up</h1>
        <hr />
        <form on:submit|preventDefault={signup}>
            <div class="field">
                <label class="label">Email</label>
                <div class="control">
                    <input type="text" bind:value={email} class="input" required class:is-danger={errorMessage} />
                    {#if errorMessage}
                        <p class="help is-danger">{errorMessage}</p>
                    {/if}
                </div>
            </div>
            <div class="field">
                <label class="label">Password</label>
                <div class="control">
                    <input type="password" bind:value={password} class="input" required />
                </div>
            </div>
            <div class="control">
                <input type="submit" class="button is-info is-light" value="Submit" />
            </div>
        </form>
        <hr />
        <p>
            Do you have an account ?
            <a href="#/login">Log in here</a>
        </p>
        <hr />
    </div>

</div>
