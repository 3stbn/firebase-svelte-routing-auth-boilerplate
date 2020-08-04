<script>
    import { FirebaseApp, User } from 'sveltefire'
    import Router, { wrap, push } from 'svelte-spa-router'
    import Home from './pages/Home.svelte'
    import Dashboard from './pages/Dashboard.svelte'
    import Signup from './pages/Signup.svelte'
    import Login from './pages/Login.svelte'
    import Navbar from './components/Navbar.svelte'
    import Loading from './components/Loading.svelte'
    import { onMount, onDestroy } from 'svelte'
    import { user, creatingUser } from './stores'
    import firebase from 'firebase/app'
    import 'firebase/firestore'
    import 'firebase/auth'

    const firebaseConfig = {
        apiKey: 'AIzaSyC-QppD-hOn-UQg7rTus994kQGmlPiwceI',
        authDomain: 'fir-auth-flow-example.firebaseapp.com',
        databaseURL: 'https://fir-auth-flow-example.firebaseio.com',
        projectId: 'fir-auth-flow-example',
        storageBucket: 'fir-auth-flow-example.appspot.com',
        messagingSenderId: '595494479084',
        appId: '1:595494479084:web:2aaa6a62f5b7d9dd6eb1c9',
    }

    firebase.initializeApp(firebaseConfig)
    const db = firebase.firestore()
    const auth = firebase.auth()

    const usersDocRef = db.collection('users')

    let loading = true
    let unsubscribe

    onMount(() => {
        unsubscribe = auth.onAuthStateChanged(async (userFirebase) => {
            if (userFirebase) {
                $user = { uid: userFirebase.uid }
                if (!$creatingUser) {
                    const doc = await usersDocRef.doc(userFirebase.uid).get()
                    const userFirestore = doc.data()
                    const { additional } = userFirestore
                    $user = { uid: userFirebase.uid, additional }
                }
            }
            loading = false
        })
    })
    onDestroy(() => {
        unsubscribe()
    })

    const routes = {
        '/': wrap(Home, { reason: 'authenticated' }, () => !$user),
        '/dashboard': wrap(Dashboard, { reason: 'unauthenticated' }, () => $user),
        '/signup': wrap(Signup, { reason: 'authenticated' }, () => !$user),
        '/login': wrap(Login, { reason: 'authenticated' }, () => !$user),
    }

    function conditionsFailed(event) {
        const { reason } = event.detail.userData
        switch (reason) {
            case 'unauthenticated':
                return push('/login')
            case 'authenticated':
                return push('/dashboard')
        }
    }
</script>

<style>
    .loading-container {
        max-width: 500px;
        display: flex;
        min-height: 100vh;
        align-content: center;
        margin: auto;
    }
</style>

<FirebaseApp {firebase}>
    {#if loading}
        <div class="loading-container">
            <Loading />
        </div>
    {:else}
        <Navbar />
        <Router {routes} on:conditionsFailed={conditionsFailed} />
    {/if}

</FirebaseApp>
