# opencode ↔ Home Assistant via MCP (ha-mcp)

## Objectif
Piloter Home Assistant (lecture/écriture des dashboards, automatisations, entités)
depuis opencode via le serveur MCP **ha-mcp** (custom component HACS).

## Prérequis
- Home Assistant avec le custom component **ha-mcp** (HACS)
- Serveur MCP exposé en HTTP sur le **LAN** (port 9583)
- URL de la forme `http://192.168.1.9:9583/private_<SECRET>` — **le chemin contient un secret**

## Config opencode
Fichier: `~/.config/opencode/opencode.json`

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "homeassistant": {
      "type": "remote",
      "url": "http://192.168.1.9:9583/private_<SECRET>",
      "enabled": true,
      "oauth": false
    }
  }
}
```

## Réseau
- Accès **LAN direct** (`192.168.1.9:9583`) → **aucun VPN nécessaire** pour opencode.
- L'URL publique HA (`https://<duckdns>:1111`) est **bloquée hors France** (geo-block) ;
  pour un accès distant, sortir via un VPN **FR**.

## Sécurité
- L'URL MCP contient un **secret** → ne pas committer / publier.
- Ancien **Long-Lived Access Token** HA (utilisé en test) : à **révoquer**
  (Profil → Sécurité → Long-Lived Access Tokens).
- **Ne pas** stocker le token dans `.bashrc` ni dans un shell history.

## Vérifs (JSON-RPC)
```bash
U="http://192.168.1.9:9583/private_<SECRET>"

# liste des outils (attendu: ~77)
curl -s -X POST "$U" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Notes
- Réponse en SSE : préfixe `data: ` → filtrer avec `sed 's/^data: //'`.
- Écritures HA (« best-practices gated ») : lire `ha_get_skill_guide` puis renvoyer
  la clé `BestPracticeKey` en cours (tourne chaque heure) + `MandatoryBPS=false`.
- Édition de dashboards : `ha_config_get_dashboard` (récupère `config_hash`) puis
  `ha_config_set_dashboard` avec `python_transform` (édition ciblée).
