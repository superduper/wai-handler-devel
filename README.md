# wai-handler-devel

This is a `wai-handler-devel` backport to support latest libraries. Original one is no longer supported. 
Reason for that was my attempt to make `wai-handler-devel` work with latest `scotty-web`. 

I fixed some compat issues and hardcoded GHC option that adds `src` dir to source path, updated versions. 

#### UPDATE: Sorry, auto reload is still broken :(

### How do I configure project to use it?
```bash
# 0. Make a cabal project
cabal init

# 1. Add wai-handler-devel to cabal list
vim project.cabal 

# 2. Init cabal sandbox 
cabal sandbox init

# 3. Install cabalg 
cabal install cabalg

# 4. Install wai-handler-devel with cabalg: 
cabalg https://github.com/superduper/wai-handler-devel

# 5. [Optional] Install transformers == 0.3.0.0,
#               This is required because of hint's dependency on ghc 
cabal install transformers == 0.3.0.0

# 6. Now we can install all dependencies together: 
cabal install --only-dependencies
```

### How do I configure scotty to run autoreload? 

Let's take as an example simplest scotty server. Save it in src/Main.hs.

```haskell
{-# LANGUAGE OverloadedStrings #-}

import Web.Scotty
import Data.Monoid (mconcat)

main = scotty 3000 $ do
  get "/:word" $ do
    beam <- param "word"
    html $ mconcat ["<h1>Scotty, ", beam, " me up!</h1>"]
```

Now we'll separate routes from application into a separate binding

```haskell
{-# LANGUAGE OverloadedStrings #-}

import Web.Scotty
import Data.Monoid (mconcat)

main = scotty 3000 routes
routes = 
  get "/:word" $ do
    beam <- param "word"
    html $ mconcat ["<h1>Scotty, ", beam, " me up!</h1>"]
```

Almost there! We add a `WAI` compatible app runner as an alternative:

```haskell
{-# LANGUAGE OverloadedStrings #-}

import Web.Scotty

import Data.Monoid (mconcat)

main = scotty 3000 routes
routes = 
  get "/:word" $ do
    beam <- param "word"
    html $ mconcat ["<h1>Scotty, ", beam, " me up!</h1>"]
dev = scottyApp routes
```

Finally we can run our example server with autoreload: 

```bash
cabal exec wai-handler-devel 3000 src/Main dev
```
You may of course change `Main` to other module name, `dev` to other bindig, and use other than `3000` port to listen at..
