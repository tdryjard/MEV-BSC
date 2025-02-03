Je vais vous guider en détail pour la configuration du projet avec bloXroute.
1. Configuration Principale (src/config.py)

bsc_local_to_gcp_config = Config(
    ### Endpoint RPC BSC (utilisé dans main.py ligne 152)
    http_endpoint="https://nd-XXX-XXX-XXX.p2pify.com/YOUR_KEY",
    
    ### Endpoint WebSocket bloXroute (utilisé pour stream_new_block_and_pending_txs)
    bloxroute_ws_endpoint="wss://virginia.bsc.blxrbdn.com/ws",
    
    ### Clé d'autorisation bloXroute (nécessaire pour l'API bloXroute)
    bloxroute_authorization="VOTRE_CLE_BLOXROUTE",
    
    ### Votre adresse BSC (pour recevoir les profits)
    account_address="0xVOTRE_ADRESSE",
    
    ### Clé privée du wallet
    account_private_key="0xVOTRE_CLE_PRIVEE",
    
    ### Adresse du contrat MEV déployé
    contract_address="0xVOTRE_CONTRAT_MEV",
    
    ### Adresse WBNB (fixe sur BSC)
    wrapped_native_token_address="0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c",
    
    ### Configuration factory pour les DEX supportés
    factory_config={
        "getPair": {
            "PANCAKESWAP_V2": ["0xcA143Ce32Fe78f1f7019d7d551a6402fC5350c73"],
            "BISWAP_V2": "0x858E3312ed3A876947EA49d572A7C42DE08af7EE",
            # Autres DEX selon besoin
        }
    },

    ### Config CoinMarketCap pour récupérer les pools
    coinmarketcap_config=CoinMarketCapConfig(
        url="https://pro-api.coinmarketcap.com/v1/cryptocurrency/listings/latest",
        dex_query_map={
            "PANCAKESWAP_V2": "pancakeswap-v2",
            "BISWAP_V2": "biswap",
            # Autres mappings
        },
        min_txns24h=5  # Filtre les pools avec peu de transactions
    )
)

## 2. Comment obtenir les différentes clés

### Endpoint RPC BSC:
- S'inscrire sur GetBlock[https://getblock.io/] ou Chainstack[https://chainstack.com/]
- Créer un nœud BSC dédié
- Copier l'URL RPC

### bloXroute:
- S'inscrire sur bloXroute[https://bloxroute.com/]
- Souscrire au plan MEV-BSC (environ 5000$/mois)
- Dans le dashboard, récupérer:
  - WebSocket Endpoint
  - Authorization Key

### Wallet BSC:
- Créer un nouveau wallet avec MetaMask
- Exporter la clé privée
- Financer le wallet avec au moins 1-2 BNB

### CoinMarketCap:
- S'inscrire sur CoinMarketCap[https://coinmarketcap.com/api/]
- Obtenir une clé API gratuite


## 3. Configuration de la stratégie

Pour configurer les paires de trading dans src/crawling.py:

async def get_pool_list_from_coinmarketcap(config):
    # Définir vos paires préférées
    preferred_pairs = [
        "WBNB/BUSD",
        "WBNB/USDT",
        "CAKE/WBNB",
        # Ajoutez d'autres paires
    ]
    
    # Définir les DEX à surveiller
    dex_list = [
        "PANCAKESWAP_V2",
        "BISWAP_V2",
        "APESWAP",
        # Ajoutez d'autres DEX
    ]


## 4. Configuration des seuils de rentabilité

Dans main.py (pour stratégie arbitrage) ou dans main_sandwich.py (pour stratégie sandwich)

#### Seuil minimum de profit (en BNB)
MIN_PROFIT_THRESHOLD = 0.001  # environ 0.3$

##### Seuil maximum de gas
MAX_GAS_MULTIPLIER = 1.5

##### Nombre de workers
worker_num = 6  # Ajustez selon votre CPU


## 5. Démarrage du bot

##### Installation
pip install -r requirements.txt

##### Démarrage
python main_sandwich.py


## Autres configs : 

### Multiprocessing (main.py ligne 158-160):
    queue = multiprocessing.Queue(maxsize=30)  # Taille max de la queue
    future_accessible_block_number = multiprocessing.Value("i", 0)
    worker_num = 6  # Nombre de workers


### Seuils de rentabilité (main.py ligne 76-77):
    ### Vérification de la rentabilité
    if arbitrage_attack.gas_used * gas_price * 1.5 < arbitrage_attack.revenue_based_on_eth:
        # ...

    ### Frais minimum pour le bundle (ligne 87)
    bundle_fee = 0.0004 * 10**18  # ~0.0004 BNB


### Gas Price (main.py ligne 97-98):
    rand_gas_price = random.randint(35 * 10**9, 45 * 10**9)
    gas_price = max_gas_price_by_tx


### Logs (main.py ligne 39-44):
    logger.add(
        "logs/arbitrage_{time:YYYY-MM-DD}.log",
        rotation="1 day",
        retention="30 days",
        level="INFO"
    )



---

## Points importants et recommandations :

#### Pour bloXroute:
- Utiliser un endpoint US East (virginia) pour minimiser la latence
- Configurer les validateurs bloXroute via leur API

#### Pour les DEX:
- Commencer avec PancakeSwap V2 qui a le plus de liquidité
- Ajouter progressivement d'autres DEX selon leur volume

#### Pour le monitoring:
- Les logs sont dans logs/arbitrage_DATE.log
- Surveiller particulièrement les messages "Arbitrage attack found" et "Not profitable"

---

1. Sélection des tokens:
- Concentrez-vous sur les paires avec forte liquidité
- WBNB/BUSD et WBNB/USDT sont les plus sûres
- Évitez les tokens avec des taxes ou des mécanismes anti-bot

2. Gestion des risques:
- Commencez avec de petits montants (0.1-0.2 BNB)
- Surveillez les logs pour les erreurs
- Mettez en place des stops automatiques

3. Optimisation:
- Utilisez un serveur proche des nœuds BSC (US East ou Europe)
- Minimum 8GB RAM, CPU 4 cœurs
- Connection internet stable et rapide

4. Monitoring:
- Les logs sont dans logs/sandwich_DATE.log
- Surveillez vos profits via BSCScan[https://bscscan.com/]
- Utilisez EigenPhi pour analyser vos transactions
