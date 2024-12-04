<script>
    import { signInAnonymously, signInWithPopup } from "firebase/auth";
    import { RecaptchaVerifier, signInWithPhoneNumber } from "firebase/auth";
    import { SignedIn, SignedOut } from "sveltefire";
    import { FacebookAuthProvider } from "firebase/auth";
    import { GoogleAuthProvider } from "firebase/auth";
    import { GithubAuthProvider } from "firebase/auth";
    import { browser } from "$app/environment";

    const GHprovider = new GithubAuthProvider();
    const Gprovider = new GoogleAuthProvider();
    const FBprovider = new FacebookAuthProvider();

    let signedinuser = {};
    let token = "";
    let solved = false;

    const do_popup = async (auth, provider, type) => {
        signInWithPopup(auth, provider)
            .then((result) => {
                // This gives you a Google Access Token. You can use it to access the Google API.
                const credential = type.credentialFromResult(result);
                token = credential.accessToken;
                signedinuser = result.user;
            })
            .catch((error) => {
                // Handle Errors here.
                const errorCode = error.code;
                const errorMessage = error.message;
                // The email of the user's account used.
                const email = error.customData.email;
                // The AuthCredential type that was used.
                const credential = type.credentialFromError(error);
                console.error("Auth error", {
                    errorCode,
                    errorMessage,
                    email,
                    credential,
                });
            });
    };

    let phoneNumber = "+1 714-555-9898";
    let verificationCode = "123456";
    let confirmationResult = null;
    let recaptchaVerifier = null;

    const beginCodeSendingProcess = async (auth) => {
        if (browser && !recaptchaVerifier && !solved) {
            recaptchaVerifier = new RecaptchaVerifier(
                "recaptcha-container",
                {
                    size: "normal",
                    callback: (response) => {
                        console.log("recaptcha solved", response);
                        solved = true;
                        doSignIn(auth);
                    },
                },
                auth
            );
            console.log("created recaptchaVerifier");
            window.recaptchaVerifier = recaptchaVerifier;
            recaptchaVerifier.render();
        } else {
            console.log("not browser");
        }
    };
    const doSignIn = async (auth) => {
        try {
            console.log("doSignIn", { auth, phoneNumber, window });
            confirmationResult = await signInWithPhoneNumber(
                auth,
                phoneNumber,
                recaptchaVerifier
            );
            console.log("back from signInWithPhoneNumber");
            const credential = confirmationResult(result);
            token = credential.accessToken;
            signedinuser = result.user;
        } catch (error) {
            console.error(error);
        }
    };

    const verifyCode = async () => {
        try {
            const credential =
                await confirmationResult.confirm(verificationCode);
            // User is now signed in, credential.user contains the user object
            console.log(credential.user);
        } catch (error) {
            console.error(error);
        }
    };
</script>

<header class="space-y-4 header">
    <p class="type">Implement one of these (do you really need them all?):</p>
    <nav class="flex gap-4">
        <SignedIn let:user let:signOut>
            <p class-info>
                Hello {user.displayName ? user.displayName : "(not supplied)"}
            </p>
            <button on:click={signOut}>Sign Out</button>
        </SignedIn>

        <div class="buttons">
            <SignedOut let:auth>
                <div class="siButton">
                    <p class="type">ANONYMOUS SIGNIN</p>
                    <p class="note">
                        This is for systems like ecommerce where you save
                        activities before they create an account -- like adding
                        items to a cart, then really logging in at checkout
                    </p>
                    <button on:click={() => signInAnonymously(auth)}
                        >Sign In anonymously</button
                    >
                </div>
            </SignedOut>

            <SignedOut let:auth>
                <div class="siButton">
                    <p class="type">GOOGLE POPUP</p>
                    <button
                        on:click={() =>
                            do_popup(auth, Gprovider, GoogleAuthProvider)}
                        >Sign In with popup</button
                    >
                </div>
            </SignedOut>

            <SignedOut let:auth>
                <div class="siButton">
                    <p class="type">GITHUB POPUP</p>
                    <button
                        on:click={() =>
                            do_popup(auth, GHprovider, GithubAuthProvider)}
                        >Sign In with popup</button
                    >
                </div>
            </SignedOut>

            <SignedOut let:auth>
                <div class="siButton">
                    <p class="type">FACEBOOK POPUP</p>
                    <p class="note">Not quite working</p>
                    <button
                        on:click={() =>
                            do_popup(auth, FBprovider, FacebookAuthProvider)}
                        >Sign In with popup</button
                    >
                </div>
            </SignedOut>

            <SignedOut let:auth>
                <div class="siButton">
                    <p class="type">PHONE VERIFICATION</p>

                    <p class="note">
                        Captcha {solved ? "solved" : "not solved"}
                    </p>
                    {#if confirmationResult}
                        <input
                            type="text"
                            bind:value={verificationCode}
                            placeholder="Verification code"
                        />
                        <button on:click={verifyCode}>Verify code</button>
                    {:else}
                        <div id="recaptcha-container" />
                        <input
                            type="tel"
                            bind:value={phoneNumber}
                            placeholder="Phone number"
                        />
                        <button on:click={() => beginCodeSendingProcess(auth)}
                            >Send Verification code</button
                        >
                    {/if}
                </div>
            </SignedOut>
        </div>
    </nav>
    <p class="title">Firebase and DB for Fun and Profit</p>
</header>

<style lang="postcss">
    .buttons {
        display: flex;
        background-color: DodgerBlue;
    }

    .siButton {
        background-color: #f1f1f1;
        margin: 10px;
        padding: 20px;
        font-size: 30px;
        flex: 1 1 0px;
    }
    .type {
        color: darkred;
        font-size: 14pt;
        background: #lightblue;
    }
    .title {
        color: black;
        font-size: 24pt;
        background: #lightblue;
    }
    .note {
        color: darkred;
        font-size: 12pt;
        background: #lightblue;
        font-style: italic;
    }
    .note {
        color: black;
        font-size: 12pt;
        background: #lightblue;
    }
    .header {
        padding: 15px;
        text-align: center;
        background-color: lightblue;
        color: white;
        font-size: 30px;
    }

    /* .profile_pic {
        width: 50px;
        height: 50px;
    } */
</style>
