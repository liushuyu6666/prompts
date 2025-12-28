# Task: RBC Mastercard Statement Extraction for Counting purpose

**Role:** Financial Data Assistant
**Status:** [READY]
**Last Updated:** 2025-12-21

## 1. Configuration & Constants
* **Source Root:** `/My Drive/Personal/$_$/RBC-Statements`
* **Target Month:** `[USER_SPECIFIED_IN_CHAT]`
    * *Instruction:* Look for the specific `YYYY-MM` date provided by the user in the current chat conversation.
* **Account Type:** `Mastercard`
    * *Note:* Matches the folder name (e.g., `Mastercard`, `Cheque`).
* **Output Format:** TSV Code Block (for Copy-Paste).

## 2. Data Structure
* **Target Columns (Final Output Order):**
    1.  `Number` (Auto-increment, starting at 1)
    2.  `Time` (Transaction Date)
    3.  `Shop Name` (Mapped Name - Column C)
    4.  `Money` (Amount)
    5.  `Type` (Formula)
    6.  `From` (Static: "RBC Mastercard")
    7.  `Note` (AI Commentary or from Map)

## 3. Execution Steps

### Step 1: Retrieve Bank Statements (Upload or Drive)
* **Action:** Locate and read the correct statement files based on the `Target Month`.
    1.  **Analyze Target Month:** Identify the `Target Month` (YYYY-MM) requested by the user in the chat.
    2.  **Determine Required Files:**
        * To capture a full calendar month, you generally need **two** statement files:
            * **File A:** Statement issued in `Target Month` (e.g., `...2025-11...`).
            * **File B:** Statement issued in `Next Month` (e.g., `...2025-12...`).
    3.  **Source Selection (Priority Order):**
        * **Option A: User Uploads (High Priority):**
            * Check the list of files uploaded to the chat.
            * **Filter:** specific instruction to **Ignore** files that do not contain the date patterns for File A or File B. Select *only* the two relevant PDFs.
        * **Option B: Google Drive (Fallback):**
            * **Tool Directive:** You have explicit permission to access the user's Google Drive. **Do not ask the user to upload the file.**
            * If no relevant files are uploaded, search the `Source Root` in Drive.
            * *Construct Paths:* `{Source Root}/{Year}/{Account Type}/MasterCard Statement-xxxx [YYYY-MM]-*.pdf`
* **Execution:** Fetch the text content of the selected (filtered) PDF files.
* **Filter (Crucial):** During extraction in Step 2, you must filter transactions to ensure their **Transaction Date** falls strictly within the `Target Month`.

### Step 2: Data Extraction & Processing
* **Action:** Parse the PDF text and loop through each transaction line.
* **Processing Rules:**
    1.  **Number:** Start at `1` and increment for each valid transaction found.
    2.  **Time:** Extract the **Transaction Date** (First date column).
    3.  **Shop Name (Matching Algorithm):**
        * Extract the raw description from the PDF line.
        * **Scan the Map:** Iterate through the **"Appendix: Map"**.
        * **Check Condition:** Does the raw description **CONTAIN** the `SEARCH_KEYWORD` from the map?
        * **If Match Found:**
            * Use the corresponding `shop name list` and `note`.
        * **If No Match:**
            * **Draft Name:** Create a reasonable name (Title Case).
            * **Flag:** Mark it as "New Merchant".
    4.  **Money:** Extract Amount.
    5.  **Type (Formula):**
        * **ALWAYS** output this specific formula for **EVERY** row:
        * `=IF(ISBLANK(INDIRECT("C"&ROW())),"",INDEX(overview!B:B,MATCH(INDIRECT("C"&ROW()),overview!A:A, 0)))`
    6.  **From:** Set to "RBC Mastercard".

### Step 3: Integrity Check (The Whitelist Validator)
* **Action:** Perform a strict final validation before generating Output 1.
* **Logic:**
    1.  For every transaction line, look at the `Shop Name` you are about to output.
    2.  **Verification:** Search for this **exact string** in the `shop name list` column of the "Appendix: Map".
    3.  **Strict Rule (Zero Tolerance):**
        * **Condition:** Is the name found in the list?
        * **YES:** Proceed.
        * **NO:** The match failed. You **MUST** change the `Shop Name` to start with `* ` (e.g., `* The Best Shop`) and set the `Note` to "New Merchant".
    4.  **Anti-Hallucination:** Do not assume a name is "correct" just because it looks readable. If it is not in the Appendix, it is **wrong** until flagged with `*`.

### Step 4: Formatting & Output
* **Output 1: Transaction Table (TSV)**
    * Generate a code block containing **Tab-Separated Values (TSV)**.
    * **Format:** Raw text where columns are separated by a single tab (`\t`) and rows by a newline.
    * **Constraint (CRITICAL):** Do **NOT** include any citation tags (like `[cite]`, ``) or footnotes inside the code block. Output **clean, raw text only**.
    * **Sorting:** Chronological (Oldest to Newest).
    * **Deliverable:**
        ```tsv
        Number	Time	Shop Name	Money	Type	From	Note
        1	MMM DD	Mapped Name	0.00	=IF(...)	RBC Mastercard	
        2	MMM DD	* New Place	0.00	=IF(...)	RBC Mastercard	New Merchant
        ```

* **Output 2: New Map Entries (Conditional)**
    * **Trigger:** If *any* "New Merchants" were found (items marked with `*`).
    * **Format:** Raw Markdown Table Rows (No header).
    * **Content:** List **ONLY** the new rows to be appended to the Appendix.
    * **Columns:** `| SEARCH_KEYWORD | shop name list | type | note |`
    * **Instruction:** Use the uppercase raw text (or a clear unique part of it) as the `SEARCH_KEYWORD`.
    * **Deliverable:**
        ```text
        | RAW KEYWORD | * New Shop Name | [Category] | |
        | ANOTHER KEY | * Another Shop | [Category] | |
        ```

* **Output 3: Daily Summary (Testing)**
    * **Purpose:** Validation to ensure all transactions were captured per date.
    * **Format:** Markdown Table.
    * **Content:** Group extracted transactions by `Time`.
    * **Columns:**
        1.  `Date`
        2.  `Count` (Total items found for this date)
        3.  `Amounts` (List of values, e.g., `[10.00, 5.50]`)
    * **Deliverable:**
        ```markdown
        ### Testing: Daily Summary
        | Date | Count | Amounts |
        |---|---|---|
        | MMM DD | 2 | [12.50, 4.00] |
        | MMM DD | 1 | [45.00] |
        ```

## Appendix: Map
* **Reference Table:** Use `ACTIVITY DESCRIPTION` as a search keyword to find the correct `shop name list`.

| SEARCH_KEYWORD | shop name list | type | note |
|:---|:---|:---|:---|
| LOBLAW | Loblaw | grocery |  |
| MCDONALD | McDonald's | restaurant |  |
| AMAZON WEB SERVICES | Amazon web services | e-service |  |
| SHOPPERS | Shoppers | grocery |  |
| CHATIME | Chatime | restaurant |  |
| DOORDASH | DoorDash | restaurant |  |
| T&T | T&T | grocery |  |
| FANTUAN | FanTuan | restaurant |  |
| HMART | hmart | grocery |  |
| FARM BOY | FARM BOY | grocery |  |
| METRO | METRO | grocery |  |
| STAPLES | STAPLES | device |  |
| FIDO | FIDO | e-service |  |
| PRESTO | PRESTO | transportation |  |
| AMAZON.CA PRIME | Amazon.ca Prime | e-service |  |
| APPLE | APPLE | device |  |
| MOGOUYAN NOODLES | MOGOUYAN noodles | restaurant |  |
| GUSCHLBAVER | Guschlbaver | restaurant |  |
| YIFANG FRUIT TEA | YiFang Fruit Tea | restaurant |  |
| MEDIUM MONTHLY | Medium corporation | e-service |  |
| RITUAL | ritual | restaurant |  |
| MONKEY SUSHI | Monkey Sushi | restaurant |  |
| ZSECURITY | ZSECURITY | e-service |  |
| NIKE | Nike | sports |  |
| LULULEMON | LuLuLemon | sports |  |
| BENKEI HIME | BENKEI HIME | restaurant |  |
| BUK CHANG DONG | BUK CHANG DONG | restaurant |  |
| FIRKIN | FIRKIN | restaurant |  |
| ARTISAN PLUS | artisan plus | restaurant |  |
| COCO | coco | restaurant |  |
| CHIPOTLE | Chipotle | restaurant |  |
| ORIGIN | Origin | entertainment |  |
| AMAZON ONLINE SHOPPING | amazon online shopping | grocery |  |
| DOLLARAMA | dollarama | grocery |  |
| 7-ELEVEN | 7-eleven | grocery |  |
| SSAM | SSAM | grocery |  |
| COSTCO | COSTCO | living |  |
| CARE&FAMILY HEALTH | care&family health | health |  |
| CHICK-FIL-A | chick-fil-a | restaurant |  |
| BDBBUY | BDBBUY | grocery |  |
| UBER EAT | uber eat | restaurant |  |
| MONGA CHICKEN | MONGA CHICKEN | restaurant |  |
| SZECHUAN EXPRESS | szechuan express | restaurant |  |
| TIM HORTONS | tim hortons | restaurant |  |
| IKEA | IKEA | living |  |
| AIRBNB | airbnb | transportation |  |
| RIDER EXPRESS | rider express | transportation |  |
| THAILAND | thailand | restaurant |  |
| CEST WHAT | cest what | restaurant |  |
| BUBBLE TEAM | bubble team | restaurant |  |
| GASPARD | GASPARD | grocery |  |
| KINTON | Kinton | restaurant |  |
| MY ROTI PLACE | my roti place | restaurant |  |
| STARBUCKS | starbucks | restaurant |  |
| CANADA POST | canada post | e-service |  |
| AROMA | aroma | restaurant |  |
| CINEPLEX | cineplex | entertainment |  |
| GOOGLE WORKSPACE | google workspace | e-service |  |
| UBERTRIP | uber trip | transportation |  |
| CORA | cora | restaurant |  |
| SHANGHAI WONTON | shanghai wonton | restaurant |  |
| CLOCKTOWER | clocktower | restaurant |  |
| KOWLOON | kowloon | grocery |  |
| OTHER RESTAURANT | other restaurant | restaurant |  |
| CONCERT TICKET | concert ticket | entertainment |  |
| LCBO | LCBO | grocery |  |
| MUJI | muji | living |  |
| LAMOUR BEAUTY | L'AMOUR BEAUTY | grocery |  |
| OTHER ENTERTAINMENT | other entertainment | entertainment |  |
| OTHER GROCERY | other grocery | grocery |  |
| UBER PASS | uber pass | restaurant |  |
| UNIQLO | uniqlo | living |  |
| ONLYFANS | onlyfans | entertainment |  |
| ALEXANDROS | alexandros | restaurant |  |
| NINTENDO GAMES | nintendo games | entertainment |  |
| TSUJIRI TEA HOUSE | tsujiri tea house | restaurant |  |
| E-TRANSFER | e-transfer | others |  |
| GYUBEE | gyubee | restaurant |  |
| EBLUE | EBlue | entertainment |  |
| COSTCO ANNUAL RENEWAL | costco annual renewal | living |  |
| AMAZON ONLINE SHOPPING (LIVING) | amazon online shopping (living) | living |  |
| JAPANESE MEDICINE | japanese medicine | living |  |
| LUCKY MOOSEFOOD MART | lucky moosefood mart | grocery |  |
| OTHER DEVICE | other device | device |  |
| THAI BOWL | thai bowl | restaurant |  |
| KFC TACO | KFC TACO | restaurant |  |
| MACHI MACHI | machi machi | restaurant |  |
| METERGY | metergy | living |  |
| XBOX GAME PASS | xbox game pass | entertainment |  |
| BEST BUY (ENTERTAINMENT) | best buy (entertainment) | entertainment |  |
| ROBOT PAYMENT TOKYO | ROBOT PAYMENT TOKYO | entertainment |  |
| WALMART (ENTERTAINMENT) | walmart (entertainment) | entertainment |  |
| WALMART (LIVING) | walmart (living) | living |  |
| WALMART (GROCERY) | walmart (grocery) | grocery |  |
| XBOX GAMES | xbox games | entertainment |  |
| APPLE (SHOP) | APPLE (shop) | e-service |  |
| PS GAME | PS game | entertainment |  |
| PIZZAIOLO | PIZZAIOLO | restaurant |  |
| CANADIAN TIRE | canadian tire | living |  |
| HOMEMADE RAMEN | homemade ramen | restaurant | 兰州拉面 |
| IKEA FOOD | ikea food | living |  |
| BET365 | bet365 | entertainment | gambling |
| CASH ADVANCE FEE | CASH ADVANCE FEE | e-service |  |
| MEDICINE | medicine | health |  |
| SHIPPING | shipping | living |  |
| NBX | NBX | grocery | 华泰 |
| YOUTUBE MEMBER | youtube member | e-service |  |
| B2 | B2 | living | boots |
| CELPIP | celpip | living |  |
| AMAZON ONLINE SHOPPING (GAME) | amazon online shopping (game) | entertainment |  |
| OINP | OINP | others |  |
| ADOBE | Adobe | e-service |  |
| UNCLE TETSU | Uncle Tetsu | restaurant |  |
| DAGU RICE | Dagu Rice | restaurant |  |
| DRIVING LICENSE | Driving License | living |  |
| SOCIAL BLEND | Social Blend | restaurant |  |
| HI YOGURT | Hi yogurt | restaurant |  |
| TURO | Turo | transportation |  |
| PETROCAN | PETROCAN | transportation | Petro Canada |
| TRIP OR TRAVEL | trip or travel | transportation |  |
| PARKS RESERVATION | parks reservation | entertainment |  |
| SUBWAY | subway | restaurant |  |
| BLUE HERON CRUISES | blue heron cruises | entertainment |  |
| UP EXPRESS | UP EXPRESS | transportation |  |
| FRESHWAY FOODMART | freshway foodmart | grocery |  |
| BRUCE PENINSULA NATION | BRUCE PENINSULA NATION | entertainment |  |
| LANZHOU LAMIAN | LanZhou lamian | restaurant |  |
| REXALL | REXALL | grocery |  |
| MID JOURNEY INC | MID JOURNEY INC | e-service |  |
| SQ MUMUSO ELT | SQ MUMUSO ELT | grocery |  |
| SP KELSO LAVENDER | SP KELSO LAVENDER | entertainment |  |
| HASTY MARKET | HASTY MARKET | grocery |  |
| CAMPING | Camping | entertainment |  |
| CHATGPT | ChatGPT | e-service |  |
| PARKING | parking | transportation |  |
| INDIGO | indigo | others |  |
| REFUEL | refuel | transportation |  |
| PR IMMIGRATION | PR immigration | e-service |  |
| AMAZON ONLINE SHOPPING (TRIP) | amazon online shopping (trip) | transportation |  |
| SHOW / CINEMA | show / cinema | entertainment |  |
| GALLERIA FRESH EXPRESS | Galleria Fresh Express | grocery |  |
| UDEMY | UDemy | e-service |  |
| BINGZ | Bingz | restaurant |  |
| KAMIYA | Kamiya | living |  |
| FUWA FUWA | fuwa fuwa | restaurant |  |
| KOSAM RESTAURANT | kosam restaurant | restaurant |  |
| POPEYE | popeye | restaurant |  |
| SANSOTEI RAMEN | sansotei ramen | restaurant |  |
| THE GREEN ISLE | The green Isle | restaurant |  |
| PLAYSTATION NETWORK | playstation network | entertainment |  |
| HOU KEE KNUNCKLE | hou kee knunckle | restaurant |  |
| LINKEDIN PRE | linkedIn Pre | e-service |  |
| OVERDRAFT HANDLING FEE | overdraft handling fee | e-service |  |
| GREEN VALLEY PR | green valley pr | grocery | 路边摊 |
| MONTHLY FEE | monthly fee | e-service |  |
| OMNI NOODLE | omni noodle | restaurant |  |
| KREAM | Kream | restaurant |  |
| GEORGES INDEPENDENT | George's independent | grocery |  |
| PORK KNUCKLES | pork knuckles | restaurant |  |
| SANGJI | sangji | restaurant |  |
| MIGRATION STUFFS | migration stuffs | others |  |
| INSTACART SUBSCRIBE | instacart subscribe | e-service |  |
| KIBO | kibo | restaurant |  |
| WENDI | wendi | restaurant |  |
| AIR CANADA | air canada | transportation |  |
| LOUIS VUITTON GROCERY | Louis Vuitton Grocery | grocery |  |
| LOUIS VUITTON LIVING | Louis Vuitton Living | living |  |
| SHANGHAI 360 | Shanghai 360 | restaurant |  |
| BUY FOR HUGO | buy for Hugo | living |  |
| CHAHALO | chahalo | restaurant |  |
| SQ *LUNA BAKERY | SQ *LUNA BAKERY | restaurant |  |
| STREET MARKET | street market | grocery |  |
| ESPRESSO HOUSE | espresso house | restaurant |  |
| HANA DON | hana don | restaurant |  |
| OTHER LIVING | other living | living |  |
| RC COFFEE | RC Coffee | restaurant |  |
| DE MELLO | DE MELLO | restaurant |  |
| 黄萌鸡 | 黄萌鸡 | restaurant |  |
| ADIDAS | adidas | living |  |
| ANNS DONBURI | ann's donburi | restaurant |  |
| CPA | CPA | e-service |  |
| BLOOM | Bloom | restaurant |  |
| JOLLIBEE | jollibee | restaurant |  |
| BAOZI 包子 | baozi 包子 | restaurant |  |
| SUKHO THAI | sukho thai | restaurant |  |
| PASTRY HOUSE | pastry house | restaurant |  |
| FRESH BUY MARKET | Fresh Buy Market | grocery |  |
| GOOD CATCH CAFE TORONTO ON | GOOD CATCH CAFE TORONTO ON | restaurant |  |
| HARBOURFRONT CAFE TORONTO ON | HARBOURFRONT CAFE TORONTO ON | restaurant | 1 yonge street, ground floor |
| CASA LOMA | CASA LOMA | entertainment |  |
| SCADDABUSH ITALIAN KITCHEN | Scaddabush Italian Kitchen | restaurant |  |
| UNKNOWN | unknown | living |  |
| DR. THOMAS VOUTSAS | DR. Thomas Voutsas | living |  |
| KINDLE AMAZON | kindle amazon | entertainment |  |
| CANADA LIFE | canada life | living |  |
| ZENQ | ZENQ | restaurant |  |
| EGG CLUB | egg club | restaurant |  |
| VILLA MADINA DIN | VILLA MADINA DIN | restaurant |  |
| TST-ROYWOODS- UNION ST TORONTO ON | TST-ROYWOODS- UNION ST TORONTO ON | restaurant |  |
| SHINTA | Shinta | restaurant |  |
| FARM | farm | entertainment |  |
| LEGENDS OF SPICE SCARBOROUGH | LEGENDS OF SPICE SCARBOROUGH | restaurant | 三大爷 |
| KAWARTHA DAIRY | KAWARTHA DAIRY | restaurant |  |
| MECP-ARROWHEAD PARK | MECP - Arrowhead Park | transportation |  |
| SEASON FRUIT HOUSE | SEASON FRUIT HOUSE | restaurant |  |
| UPE EXPRESS PEARSON | UPE EXPRESS PEARSON | transportation |  |
| 131 HILLSDALE AVE E RENT FEE | 131 HILLSDALE AVE E rent fee | living | 新家住址&房租 |
| OLLY FRESCOS | olly frescos | restaurant |  |
| SQ *DELYSEES LUXURY DE TORONTO ON | SQ *DELYSEES LUXURY DE TORONTO ON | restaurant |  |
| AMAZON ONLINE SHOPPING (OTHERS) | amazon online shopping (others) | others |  |
| HERO TEA | hero tea | restaurant |  |
| RAILWAY BENTO | railway bento | restaurant |  |
| CAIXIN | caixin | e-service | 财新网 |
| BELL | Bell | living |  |
| CHAIYO BY PAI | CHAIYO BY PAI | restaurant |  |
| KIMCHI | Kimchi | restaurant |  |
| STOCK TC | STOCK TC | grocery |  |
| PUBS | Pubs | restaurant | 任何酒吧 |
| SHAKE SHACK | Shake Shack | restaurant |  |
| TEMU | Temu | living |  |
| 蟹蟹 | 蟹蟹 | restaurant |  |
| WHOLE FOOD | whole food | grocery |  |
| CASH BACK | cash back | others |  |
| EATALY | EATALY | grocery |  |
| TASTE OF CHINA | Taste of China | restaurant | 苗王酸菜鱼 |
| DUKE OF KENT | Duke of KENT | restaurant |  |
| PARIS BAGUETTE | Paris Baguette | restaurant |  |
| H&M | H&M | living |  |
| GOOGLE COLAB | google colab | e-service |  |
| AJISEN | AJISEN | restaurant |  |
| OSMOWS | OSMOWS | restaurant |  |
| NESPRESSO | NESPRESSO | restaurant |  |
| MAMA LEE | MAMA LEE | restaurant |  |
| TEN MILES BBQ | TEN MILES BBQ | restaurant |  |
| JIMMY THE GREEK | JIMMY THE GREEK | restaurant |  |
| RAMEN MISOYA | ramen misoya | restaurant |  |
| ALOETTE | aloettego | restaurant |  |
| AVIVA GENERAL INSURANC | AVIVA GENERAL INSURANC | living |  |
| PROMPTCHAN AI | PROMPTCHAN AI | e-service |  |
| FLIGHT TICKET | Flight ticket | transportation |  |
| OTHERS | others | others |  |
| NORTH FACE | north face | living |  |
| TIGER TEA | tiger tea | restaurant |  |
| BOURBON ST GRILL EG TORONTO ON | BOURBON ST GRILL EG TORONTO ON | restaurant |  |
| CALIFORNIA THAI | California Thai | restaurant |  |
| BEST BUY (LIVING) | best buy (living) | living |  |
| WINE RACK | wine rack | restaurant |  |
| GRAZIE | Grazie | restaurant |  |
| PIZZA MARU | PIzza Maru | restaurant |  |
| GELATO | gelato | restaurant |  |
| MIZZICA GELATERIA | gelato | restaurant |  |
| OPENART | openart ai | e-service |  |
| RUNWAY STANDARD PLAN | runway standard plan | e-service |  |
| BEANFIELD | beanfield | living |  |
| BADMINTON | Badminton | entertainment |  |
| ROBINTIDE FARMS | ROBINTIDE FARMS | grocery | picking up in farm |
| MOBIMATTER | Mobimatter | e-service |  |
| WESTJET | WestJet | transportation |  |
| MANDARIN YONGE | Mandarin Yonge | restaurant |  |
| JESSE'S NF | Jesse's No Frills | grocery | No Frill |
| KAI WEI | kai wei | grocery | 华隆 |
| WINNERS | winners | living |  |
| SOBEYS | sobeys | grocery |  |
| ASIA FOOD MART | asia food mart | grocery | 丰亚 |
| ESSO | esso | transportation |  |
| FOOD BASICS | food basics | grocery |  |
| GROCERY IN CHINA | grocery in China | grocery |  |
| TRANSPORTATION IN CHINA | transportation in China | transportation |  |
| LIVING IN CHINA | living in China | living |  |
| RESTAURANT IN CHINA | restaurant in China | restaurant |  |
| GROCERY IN KOREA | grocery in Korea | grocery |  |
| TRANSPORTATION IN KOREA | transportation in Korea | transportation |  |
| LIVING IN KOREA | living in Korea | living |  |
| RESTAURANT IN KOREA | restaurant in Korea | restaurant |  |
| SUNNY FOODMART | sunny foodmart | grocery | |
| THE BEST SHOP | The Best Shop | grocery |  |
| CITY OF TORONTO FERRY | City Of Toronto Ferry | transportation |  |
| 99 CENT DEPOT | 99 Cent Depot | grocery |  |
| CLINE BOT INC | Cline Bot Inc | e-service |  |
| MTO TSD SO TO SHEPPARD | Mto Tsd So To Sheppard | transportation |  |
| THE ONE FUSION CUISINE | The One Fusion Cuisine | restaurant |  |
| HORNBLOWER NIAGARA CRU | Hornblower Niagara Cru | entertainment |  |
| SHACK EGLINTON | Shake Shack | restaurant | |
| GRAZIE RISTORANTE | Grazie | restaurant | |
| BINGZ CRISPY BURGER | Bingz | restaurant | |
| BOAT NOODLES | 555 boat noodles | restaurant | |
| ZARA | zara | living | |
| SPORT CHEK | sportchek | living | |
| TOMMY HILFIGER | TOMMY HILFIGER | living | |
| YMCA | YMCA | living | |
