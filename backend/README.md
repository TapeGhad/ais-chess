# Chess - backend server

![Chess board](https://baptistandreflector.org/wp-content/uploads/2017/05/chess-game.jpg "Chess, Chess, Mat...")

## Basic explonaton of front-back communication:
---

### **Authorization with login & pass:**

> + ***Frontend***

Firstly, after entering login and password is a specified fields, we create an object `userObj`:

    let userObj = {
        user: {
          login: document.getElementById("Login").value,
          password: document.getElementById("Password").value
        }
      }

and then, send this object to backend server through `axios` request:

    axios.post('/api/users/login', userObj)
        .then(async (response) => {...}, (error) => {
                            console.log(error);
                          });

> - ***Backend***

After we sent an `axios` request from front, accept it on back:

    router.post('/login', (req, res, next) => {
    const { body: { user } } = req;

    if(!user.login) {
        return res.json({
        errors: {
            login: 'is required',
        },
        });
    }

    if(!user.password) {
        return res.json({
        errors: {
            password: 'is required',
        },
        });
    }

    return passport.authenticate('local', { session: false }, (err, passportUser, info) => {
        if(err) {
        return next(err);
        }

        if(passportUser) {
        const user = passportUser;
        user.token = passportUser.generateJWT();

        return res.json({ user: user.toAuthJSON() });
        }

        return res.json({
        errors: {
            user: 'not found',
        },
        });
    })(req, res, next);
    });

as a result, if user is OK and exists in DB, create a `JWT` token and send back to front.

> + ***Frontend***

Finally, in callback function of `axios` request, setup header "Authorization" for future indicating user on the server, 
save **token**, **name** and **id** to *Vue store* & *localStorage*:

    axios.post('/api/users/login', userObj).then(async (response) => {
        if (response.data.user) {
            axios.defaults.headers.common["Authorization"] = `Token ${response.data.user.token}`;
            window.localStorage.setItem("userLog", response.data.user.token);
            window.localStorage.setItem("userName", response.data.user.name);
            window.localStorage.setItem("userID", response.data.user.id);
            this.$store.commit("setLoginUser", response.data.user.name);
            this.$store.commit("setLoginUserID", response.data.user.id);
            router.push('/account', () => {});
            this.dialog = false;
        }
        }, (error) => {
        console.log(error);
        });

---

### **Authorization with _Google account_:**

> + ***Frontend***

Here, the beggining of the authorization process is absolutely another. After user clicked a **Google** button, he is redirected to special window:

    this.$gAuth
        .signIn()

where he can choose a *Google account* to Log in. Then, with a promise we get a **GoogleUser** obj, which holds an *id token*. We need this one to send on our backend (with `axios` request):

    this.$gAuth
        .signIn()
        .then(async GoogleUser => {
          await axios.post('/api/users/google', {
                        ID: GoogleUser.getAuthResponse().id_token
          }).then( async response => {...})}

> + ***Backend***

After we sent an `axios` request from front, accept it on back, check userID (if it not exp, created for our app):

    router.post('/google', (req, res, next) => {
    const { body: { ID } } = req;

    const client = new OAuth2Client("745478166073-6pqqiojeous9m3s3moi88krc0obh6u8d.apps.googleusercontent.com");

    async function verify() {
        const ticket = await client.verifyIdToken({
            idToken: ID,
            audience: "745478166073-6pqqiojeous9m3s3moi88krc0obh6u8d.apps.googleusercontent.com", 
        });

    const payload = ticket.getPayload();
    const userid = payload['sub'];
    const email = payload['email'];
    const name = payload['name'];

and then check if it exists if DB (if no - create one) and return info for front:

    Users.findOne({ login: userid }, async function( err, isUser) {
      if (isUser)
      {
        const token = await generateJWTGoogle(isUser._id, name);
        res.json({ user : {email, name, token, id: isUser._id }});
      }
      else {
        const userObj = {
          email,
          login: userid,
          name
        }
        const finalUser = new Users(userObj);
        finalUser.save()
        .then(() => res.json({ user: finalUser.toAuthJSON() }));
      }
    })

> + ***Frontend***

Finally, in callback function of `axios` request, setup header "Authorization" for future indicating user on the server, 
save **token**, **name** and **id** to *Vue store* & *localStorage*:

    this.$gAuth
        .signIn()
        .then(async GoogleUser => {
          await axios.post('/api/users/google', {
            ID: GoogleUser.getAuthResponse().id_token
            }).then(async (response) => {
                axios.defaults.headers.common["Authorization"] = `Token ${response.data.user.token}`;
                window.localStorage.setItem("userLog", response.data.user.token);
                window.localStorage.setItem("userName", response.data.user.name);
                window.localStorage.setItem("userID", response.data.user.id);
                this.$store.commit("setLoginUser", response.data.user.name);
                this.$store.commit("setLoginUserID", response.data.user.id);
                router.push('/account', () => {});
                }, (error) => {
                console.log(error);
                });

---