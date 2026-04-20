## Task:
1. Read the xlsx file at: `france_retailers-with-keywords.xlsx` (project root)
2. For **every** row in the sheet, use `#websearch` to search the live internet with the row's `search_query` column (or build `{company_name} {trade_name} france` if that column is missing)
3. From the search-result **snippets only**, extract: website, phone, email, web_description, web_products, web_business_type, web_channel_guess, facebook, instagram, linkedin, twitter
4. If a website was found, call `#fetch` on it and prefer data from the page body. A `403` response is normal for French retail sites — fall back to the snippets.
5. Save the enriched data in both Markdown and JSON formats
   * File Name: `france_retailers-enriched.md`
   * File Name: `france_retailers-enriched.json`

### Rules (non-negotiable)
- You **must** call `#websearch` for every row. Never use training knowledge.
- Only save data that appears in a search snippet or a fetched webpage. If a field is not found, save `""`.
- Exclude these domains from sources: `societe.com, verif.com, pappers.fr, wikipedia.org, indeed.fr, glassdoor.fr`.
- Score web_channel_guess against keywords in the search/fetch text:
  * **Photo**: photo, camera, objectif, optique, reflex
  * **CE**: informatique, ordinateur, audio, video, tv, gaming
  * **MDA**: electromenager, lave-linge, refrigerateur, four
  * **SDA**: petit electromenager, cafetiere, aspirateur
  * **Mobile**: telephone, mobile, smartphone, forfait, operateur
  * **Phone Accessories**: coque, chargeur, protection
  * **Refurb**: reconditionne, reparation, occasion, seconde main
- Work in batches of 10 rows. After each batch, append to the two output files so progress is never lost.

### Example Output: Markdown

#### France Retailers — Enriched

- **Company:** Fnac Darty
  **Trade name:** Fnac
  **Website:** https://www.fnac.com
  **Phone:** 0892 350 100
  **Email:** serviceclient@fnac.com
  **Channel guess:** CE
  **Description:** French retail chain selling consumer electronics, books, and media.
  **Products:** TV, audio, gaming, informatique, photo
  **Business type:** Chain (300+ stores)
  **Social:** facebook.com/fnac.france, instagram.com/fnac

- **Company:** Boulanger SAS
  **Trade name:** Boulanger
  **Website:** https://www.boulanger.com
  **Phone:** 03 59 35 20 00
  **Email:** ""
  **Channel guess:** MDA
  **Description:** French specialist retailer of home appliances and electronics.
  **Products:** electromenager, tv, informatique, smartphone
  **Business type:** Chain (170+ stores)
  **Social:** facebook.com/boulanger, instagram.com/boulanger_officiel

### Example Output: JSON
[
  {
    "company_name": "Fnac Darty",
    "trade_name": "Fnac",
    "website": "https://www.fnac.com",
    "phone": "0892 350 100",
    "email": "serviceclient@fnac.com",
    "web_description": "French retail chain selling consumer electronics, books, and media.",
    "web_products": "TV, audio, gaming, informatique, photo",
    "web_business_type": "Chain (300+ stores)",
    "web_channel_guess": "CE",
    "facebook": "https://facebook.com/fnac.france",
    "instagram": "https://instagram.com/fnac",
    "linkedin": "",
    "twitter": ""
  },
  {
    "company_name": "Boulanger SAS",
    "trade_name": "Boulanger",
    "website": "https://www.boulanger.com",
    "phone": "03 59 35 20 00",
    "email": "",
    "web_description": "French specialist retailer of home appliances and electronics.",
    "web_products": "electromenager, tv, informatique, smartphone",
    "web_business_type": "Chain (170+ stores)",
    "web_channel_guess": "MDA",
    "facebook": "https://facebook.com/boulanger",
    "instagram": "https://instagram.com/boulanger_officiel",
    "linkedin": "",
    "twitter": ""
  }
]
