# iOS Locator Strategies — FILMFOX MovieApp

**App Bundle ID:** `com.okanorkun.MovieApp`  
**Framework:** UIKit · Swift 5 · Programmatic UI (no Storyboard except DetailsMovieViewController XIB)  
**Simulator:** iPhone 17 Pro · UDID `0CEC38A6-4EAD-4D44-B8F4-DF6D8902D6FD`  
**Automation Stack:** Appium 2 + XCUITest Driver  

---

## Table of Contents
1. [Locator Strategy Overview](#1-locator-strategy-overview)
2. [Strategy Priority Matrix](#2-strategy-priority-matrix)
3. [safeID Normalisation Rule](#3-safeid-normalisation-rule)
4. [MainViewController (Home Screen)](#4-mainviewcontroller--home-screen)
5. [TopRatedViewController](#5-topratedviewcontroller)
6. [DetailsMovieViewController](#6-detailsmovieviewcontroller)
7. [Appium Capabilities Reference](#7-appium-capabilities-reference)
8. [XCUITest Native Reference](#8-xcuitest-native-reference)
9. [Localization Support](#9-localization-support)

---

## 1. Locator Strategy Overview

| Strategy | Appium `By` (Java) | XCUITest API | Speed | Reliability | Use When |
|---|---|---|---|---|---|
| **Accessibility ID** | `AppiumBy.accessibilityId()` | `.identifier` | ⚡ Fast | ✅ Highest | Element has `accessibilityIdentifier` set |
| **Class Chain** | `AppiumBy.iOSClassChain()` | N/A | ⚡ Fast | ✅ High | Need relative position or attribute filter |
| **NSPredicate String** | `AppiumBy.iOSNsPredicateString()` | N/A | ⚡ Fast | ✅ High | Complex attribute queries on native types |
| **XPath** | `By.xpath()` | N/A | 🐢 Slow | ⚠️ Fragile | Last resort — full-tree traversal |
| **Class Name** | `By.className()` | `.buttons` etc. | ⚡ Fast | ⚠️ Low | Count/index queries only |
| **ID** | `By.id()` | N/A | ⚡ Fast | ✅ High | Same as Accessibility ID in XCUITest |

---

## 2. Strategy Priority Matrix

```
Priority 1 — Accessibility ID        ~"btn_search"
Priority 2 — Class Chain              **/XCUIElementTypeButton[`name == "btn_search"`]
Priority 3 — NSPredicate String       type == "XCUIElementTypeButton" AND name == "btn_search"
Priority 4 — XPath (last resort)      //XCUIElementTypeButton[@name="btn_search"]
```

**Decision flow:**

```
Does the element have accessibilityIdentifier set?
├── YES → Use Accessibility ID  ✅
└── NO  →  Is it a unique element type + label combo?
           ├── YES → Use NSPredicate or Class Chain
           └── NO  → Use Class Chain with ancestor context
                      └── Still not unique? → XPath (add ID instead!)
```

---

### General Examples — Priority Order (not app-specific)

These are generic, reusable patterns. Apply the same priority to any iOS app.

#### Priority 1 — Accessibility ID
```java
// Fastest and most reliable — use whenever accessibilityIdentifier is set in source
import io.appium.java_client.AppiumBy;
import org.openqa.selenium.WebElement;
import java.util.List;

WebElement element  = driver.findElement(AppiumBy.accessibilityId("element_identifier"));
List<WebElement> elements = driver.findElements(AppiumBy.accessibilityId("shared_identifier"));

// XCUITest equivalent
// app.buttons["element_identifier"]
// app.cells["element_identifier"]
// app.otherElements["element_identifier"]
```

#### Priority 2 — Class Chain
```java
// Use when: no AID set, need ancestor context, or positional queries
// Syntax: **/XCUIElementType[`predicate`]/ChildType[`predicate`]

// Match by name attribute
WebElement element = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeButton[`name == \"submit_btn\"`]")
);

// Match by label (visible text)
WebElement element = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeButton[`label == \"Submit\"`]")
);

// Match by value
WebElement element = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeTextField[`value == \"Search...\"`]")
);

// Positional — second cell in a table
WebElement element = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeTable/XCUIElementTypeCell[2]")
);

// First child button inside a named container
WebElement element = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeOther[`name == \"toolbar\"`]/XCUIElementTypeButton[1]")
);

// Nested — button inside cell inside collection
WebElement element = driver.findElement(
    AppiumBy.iOSClassChain(
        "**/XCUIElementTypeCollectionView/XCUIElementTypeCell[3]/XCUIElementTypeButton[`label == \"Play\"`]"
    )
);

// Multiple predicates combined
WebElement element = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeButton[`name == \"action_btn\" AND visible == true`]")
);
```

#### Priority 3 — NSPredicate String
```java
// Use when: filtering multiple elements by attribute, no AID, complex conditions
// Operators: ==, !=, BEGINSWITH, ENDSWITH, CONTAINS, AND, OR, NOT

// Single attribute
WebElement element = driver.findElement(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeButton\" AND label == \"Submit\"")
);

// Visible only
WebElement element = driver.findElement(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeTextField\" AND visible == true")
);

// Starts with (useful for dynamic identifiers)
List<WebElement> elements = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeCell\" AND name BEGINSWITH \"row_\"")
);

// Contains text
List<WebElement> elements = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeStaticText\" AND label CONTAINS \"Error\"")
);

// Enabled state
WebElement element = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeButton\" AND enabled == true AND label == \"Login\""
    )
);

// OR condition
List<WebElement> elements = driver.findElements(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeButton\" OR type == \"XCUIElementTypeImage\""
    )
);
```

#### Priority 4 — XPath (last resort)
```java
// Use only when Priority 1-3 are not viable — slowest strategy
import org.openqa.selenium.By;

// By attribute
WebElement element = driver.findElement(By.xpath("//XCUIElementTypeButton[@name='submit_btn']"));

// By label text
WebElement element = driver.findElement(By.xpath("//XCUIElementTypeButton[@label='Submit']"));

// By index (1-based in XPath)
WebElement element = driver.findElement(By.xpath("(//XCUIElementTypeButton)[2]"));

// Child relationship
WebElement element = driver.findElement(
    By.xpath("//XCUIElementTypeTable[@name='list']//XCUIElementTypeCell[1]/XCUIElementTypeButton")
);

// Contains text
List<WebElement> elements = driver.findElements(
    By.xpath("//XCUIElementTypeStaticText[contains(@label, 'Error')]")
);
```

---

## 3. safeID Normalisation Rule

Dynamic locators are generated by normalising the movie title:

```swift
// Swift source (MainViewController.swift & CarouselMovieCell)
let safeID = title
    .lowercased()
    .replacingOccurrences(of: " ", with: "_")
    .replacingOccurrences(of: "'", with: "")
    .replacingOccurrences(of: ":", with: "")
```

**Examples:**

| Movie Title | safeID |
|---|---|
| `Inception` | `inception` |
| `Harry Potter` | `harry_potter` |
| `Schindler's List` | `schindlers_list` |
| `The Dark Knight` | `the_dark_knight` |
| `Spider-Man: No Way Home` | `spider-man_no_way_home` |

```java
// Java helper — mirrors Swift normalisation
public static String safeId(String title) {
    return title.toLowerCase()
                .replace(" ", "_")
                .replace("'", "")
                .replace(":", "");
}
```

---

## 4. MainViewController — Home Screen

**File:** `Controllers/MainViewController/MainViewController.swift`  
**Cells:** `MovieTableCell` · `CarouselMovieCell` (defined in same file)

---

### 4.1 Static Locators

#### Header & Navigation

| Element | Type | Accessibility ID | XCUIElement Type |
|---|---|---|---|
| Header view | UIView | `home_header` | XCUIElementTypeOther |
| Search button | UIButton | `btn_search` | XCUIElementTypeButton |
| Trophy button | UIButton | `btn_trophy` | XCUIElementTypeButton |
| Filter button | UIButton | `btn_filter` | XCUIElementTypeButton |
| Sort button | UIButton | `btn_sort` | XCUIElementTypeButton |
| Saved/Favorites button | UIButton | `btn_saved` | XCUIElementTypeButton |

#### Content Areas

| Element | Type | Accessibility ID | XCUIElement Type |
|---|---|---|---|
| Movies table view | UITableView | `movies_table` | XCUIElementTypeTable |
| Carousel collection view | UICollectionView | `carousel_collection` | XCUIElementTypeCollectionView |
| Section tab scroll view | UIScrollView | `section_tab_bar` | XCUIElementTypeScrollView |

#### Section Tab Buttons

| Element | Accessibility ID | XCUIElement Type |
|---|---|---|
| Trending tab | `tab_trending` | XCUIElementTypeButton |
| Romantic tab | `tab_romantic` | XCUIElementTypeButton |
| Action tab | `tab_action` | XCUIElementTypeButton |
| Top Rated tab | `tab_top_rated` | XCUIElementTypeButton |

---

### 4.2 Locator Examples — Accessibility ID

```java
// Appium Java — AppiumBy.accessibilityId()
import io.appium.java_client.AppiumBy;
import org.openqa.selenium.WebElement;

// Header
WebElement header       = driver.findElement(AppiumBy.accessibilityId("home_header"));

// Nav buttons
WebElement btnSearch    = driver.findElement(AppiumBy.accessibilityId("btn_search"));
WebElement btnTrophy    = driver.findElement(AppiumBy.accessibilityId("btn_trophy"));
WebElement btnFilter    = driver.findElement(AppiumBy.accessibilityId("btn_filter"));
WebElement btnSort      = driver.findElement(AppiumBy.accessibilityId("btn_sort"));
WebElement btnSaved     = driver.findElement(AppiumBy.accessibilityId("btn_saved"));

// Content
WebElement moviesTable    = driver.findElement(AppiumBy.accessibilityId("movies_table"));
WebElement carousel       = driver.findElement(AppiumBy.accessibilityId("carousel_collection"));
WebElement sectionTabBar  = driver.findElement(AppiumBy.accessibilityId("section_tab_bar"));

// Section tabs
WebElement tabTrending  = driver.findElement(AppiumBy.accessibilityId("tab_trending"));
WebElement tabRomantic  = driver.findElement(AppiumBy.accessibilityId("tab_romantic"));
WebElement tabAction    = driver.findElement(AppiumBy.accessibilityId("tab_action"));
WebElement tabTopRated  = driver.findElement(AppiumBy.accessibilityId("tab_top_rated"));
```

---

### 4.3 Locator Examples — Class Chain

```java
// Appium Java — AppiumBy.iOSClassChain()
// Format: **/XCUIElementType[`predicate`]

// Search button within header
WebElement btnSearchCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeButton[`name == \"btn_search\"`]")
);

// First button inside section_tab_bar scroll view
WebElement firstTabCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeScrollView[`name == \"section_tab_bar\"`]/XCUIElementTypeButton[1]")
);

// Tab by exact name
WebElement tabActionCc = driver.findElement(
    AppiumBy.iOSClassChain(
        "**/XCUIElementTypeScrollView[`name == \"section_tab_bar\"`]/XCUIElementTypeButton[`name == \"tab_action\"`]"
    )
);

// Any button in header area
List<WebElement> headerButtonsCc = driver.findElements(
    AppiumBy.iOSClassChain("**/XCUIElementTypeOther[`name == \"home_header\"`]/XCUIElementTypeButton")
);

// Movies table
WebElement moviesTableCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeTable[`name == \"movies_table\"`]")
);
```

---

### 4.4 Locator Examples — NSPredicate String

```java
// Appium Java — AppiumBy.iOSNsPredicateString()

// Any button whose name starts with "btn_"
List<WebElement> allActionBtns = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeButton\" AND name BEGINSWITH \"btn_\"")
);

// Section tabs (name starts with "tab_")
List<WebElement> allTabs = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeButton\" AND name BEGINSWITH \"tab_\"")
);

// Visible table
WebElement moviesTablePred = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeTable\" AND name == \"movies_table\" AND visible == true"
    )
);

// Carousel collection
WebElement carouselPred = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeCollectionView\" AND name == \"carousel_collection\""
    )
);
```

---

### 4.5 Locator Examples — XPath (last resort)

```java
// Appium Java — By.xpath()
import org.openqa.selenium.By;

WebElement btnSearchXp   = driver.findElement(By.xpath("//XCUIElementTypeButton[@name='btn_search']"));
WebElement btnTrophyXp   = driver.findElement(By.xpath("//XCUIElementTypeButton[@name='btn_trophy']"));
WebElement moviesTableXp = driver.findElement(By.xpath("//XCUIElementTypeTable[@name='movies_table']"));

// All section tab buttons
List<WebElement> tabsXp = driver.findElements(
    By.xpath("//XCUIElementTypeScrollView[@name='section_tab_bar']//XCUIElementTypeButton")
);
```

---

### 4.6 Dynamic Cell Locators — MovieTableCell

Pattern: `movie_card_{safeID}` · `btn_play_{safeID}` · `rating_badge_{safeID}`

```java
public static String movieCard(String title)   { return "movie_card_"   + safeId(title); }
public static String btnPlay(String title)     { return "btn_play_"     + safeId(title); }
public static String ratingBadge(String title) { return "rating_badge_" + safeId(title); }

// Usage
WebElement inceptionCard   = driver.findElement(AppiumBy.accessibilityId(movieCard("Inception")));
WebElement inceptionPlay   = driver.findElement(AppiumBy.accessibilityId(btnPlay("Inception")));
WebElement inceptionRating = driver.findElement(AppiumBy.accessibilityId(ratingBadge("Inception")));

// Class Chain — card by ID
WebElement inceptionCardCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeCell[`name == \"movie_card_inception\"`]")
);

// Play button inside a specific card via Class Chain
WebElement inceptionPlayCc = driver.findElement(
    AppiumBy.iOSClassChain(
        "**/XCUIElementTypeCell[`name == \"movie_card_inception\"`]/XCUIElementTypeButton[`name == \"btn_play_inception\"`]"
    )
);

// NSPredicate — all play buttons
List<WebElement> allPlayBtns = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeButton\" AND name BEGINSWITH \"btn_play_\"")
);

// NSPredicate — all movie cards
List<WebElement> allCards = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeCell\" AND name BEGINSWITH \"movie_card_\"")
);
```

---

### 4.7 Dynamic Cell Locators — CarouselMovieCell

Pattern: `carousel_card_{safeID}` · `carousel_play_{safeID}` · `carousel_title_{safeID}` · `carousel_rating_{safeID}`

```java
public static String carouselCard(String title)   { return "carousel_card_"   + safeId(title); }
public static String carouselPlay(String title)   { return "carousel_play_"   + safeId(title); }
public static String carouselTitle(String title)  { return "carousel_title_"  + safeId(title); }
public static String carouselRating(String title) { return "carousel_rating_" + safeId(title); }

// Usage
WebElement avengersCard   = driver.findElement(AppiumBy.accessibilityId(carouselCard("The Avengers")));
WebElement avengersPlay   = driver.findElement(AppiumBy.accessibilityId(carouselPlay("The Avengers")));
WebElement avengersTitle  = driver.findElement(AppiumBy.accessibilityId(carouselTitle("The Avengers")));
WebElement avengersRating = driver.findElement(AppiumBy.accessibilityId(carouselRating("The Avengers")));

// Class Chain — carousel card via parent collection
WebElement avengersCardCc = driver.findElement(
    AppiumBy.iOSClassChain(
        "**/XCUIElementTypeCollectionView[`name == \"carousel_collection\"`]/XCUIElementTypeCell[`name == \"carousel_card_the_avengers\"`]"
    )
);

// Class Chain — play button inside carousel card
WebElement avengersPlayCc = driver.findElement(
    AppiumBy.iOSClassChain(
        "**/XCUIElementTypeCell[`name == \"carousel_card_the_avengers\"`]/XCUIElementTypeButton[`name == \"carousel_play_the_avengers\"`]"
    )
);

// NSPredicate — all carousel cards
List<WebElement> allCarouselCards = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeCell\" AND name BEGINSWITH \"carousel_card_\"")
);

// NSPredicate — all carousel play buttons
List<WebElement> allCarouselPlays = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeButton\" AND name BEGINSWITH \"carousel_play_\"")
);
```

---

## 5. TopRatedViewController

**File:** `Controllers/TopRatedViewController/TopRatedViewController.swift`  
**Presentation:** Modal (presented from `btn_trophy` tap on Home screen)  
**Cell:** `TopRatedMovieCell` (defined in same file)

---

### 5.1 Static Locators

| Element | Type | Accessibility ID | XCUIElement Type |
|---|---|---|---|
| Back button | UIButton | `btn_back` | XCUIElementTypeButton |
| Info banner label | UILabel | `top_rated_info_banner` | XCUIElementTypeStaticText |
| Movies collection view | UICollectionView | `top_rated_collection` | XCUIElementTypeCollectionView |
| Page indicator (dots) | UIPageControl | `page_indicator` | XCUIElementTypePageIndicator |

---

### 5.2 Locator Examples — Accessibility ID

```java
WebElement btnBack       = driver.findElement(AppiumBy.accessibilityId("btn_back"));
WebElement infoBanner    = driver.findElement(AppiumBy.accessibilityId("top_rated_info_banner"));
WebElement topRatedColl  = driver.findElement(AppiumBy.accessibilityId("top_rated_collection"));
WebElement pageIndicator = driver.findElement(AppiumBy.accessibilityId("page_indicator"));
```

---

### 5.3 Locator Examples — Class Chain

```java
// Back button
WebElement btnBackCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeButton[`name == \"btn_back\"`]")
);

// Info banner
WebElement infoBannerCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeStaticText[`name == \"top_rated_info_banner\"`]")
);

// Page indicator
WebElement pageIndicatorCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypePageIndicator[`name == \"page_indicator\"`]")
);

// Top Rated collection
WebElement topRatedCollCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeCollectionView[`name == \"top_rated_collection\"`]")
);

// First cell inside top rated collection
WebElement firstCellCc = driver.findElement(
    AppiumBy.iOSClassChain(
        "**/XCUIElementTypeCollectionView[`name == \"top_rated_collection\"`]/XCUIElementTypeCell[1]"
    )
);
```

---

### 5.4 Locator Examples — NSPredicate String

```java
// Back button
WebElement btnBackPred = driver.findElement(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeButton\" AND name == \"btn_back\"")
);

// Info banner visible
WebElement infoBannerPred = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeStaticText\" AND name == \"top_rated_info_banner\" AND visible == true"
    )
);

// Page indicator
WebElement pageIndicatorPred = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypePageIndicator\" AND name == \"page_indicator\""
    )
);
```

---

### 5.5 Dynamic Cell Locators — TopRatedMovieCell

Pattern: `top_rated_card_{rank}` · `btn_watch_now_{rank}` · `top_rated_title_{rank}` · `top_rated_rating_{rank}`  
**rank** starts at `1` (1-indexed).

```java
public static String topRatedCard(int rank)  { return "top_rated_card_" + rank; }
public static String btnWatchNow(int rank)   { return "btn_watch_now_"  + rank; }
public static String topRatedTitle(int rank) { return "top_rated_title_" + rank; }
public static String topRatedRating(int rank){ return "top_rated_rating_" + rank; }

// Usage — rank 1
WebElement card1   = driver.findElement(AppiumBy.accessibilityId(topRatedCard(1)));
WebElement watch1  = driver.findElement(AppiumBy.accessibilityId(btnWatchNow(1)));
WebElement title1  = driver.findElement(AppiumBy.accessibilityId(topRatedTitle(1)));
WebElement rating1 = driver.findElement(AppiumBy.accessibilityId(topRatedRating(1)));

// Class Chain — Watch Now button inside rank 1 card
WebElement watchNow1Cc = driver.findElement(
    AppiumBy.iOSClassChain(
        "**/XCUIElementTypeCell[`name == \"top_rated_card_1\"`]/XCUIElementTypeButton[`name == \"btn_watch_now_1\"`]"
    )
);

// Class Chain — title label inside rank 3 card
WebElement title3Cc = driver.findElement(
    AppiumBy.iOSClassChain(
        "**/XCUIElementTypeCell[`name == \"top_rated_card_3\"`]/XCUIElementTypeStaticText[`name == \"top_rated_title_3\"`]"
    )
);

// NSPredicate — all top rated cards
List<WebElement> allTopCards = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeCell\" AND name BEGINSWITH \"top_rated_card_\"")
);

// NSPredicate — all Watch Now buttons
List<WebElement> allWatchNow = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeButton\" AND name BEGINSWITH \"btn_watch_now_\"")
);

// XPath — Watch Now for specific rank
WebElement watchNow2Xp = driver.findElement(
    By.xpath("//XCUIElementTypeButton[@name='btn_watch_now_2']")
);
```

---

## 6. DetailsMovieViewController

**File:** `Controllers/DetailsMovieViewController/DetailsMovieViewController.swift`  
**Presentation:** XIB-based (`DetailsMovieViewController.xib`) — pushed from movie cell tap  

### 6.1 Known UI Elements (IBOutlets)

| IBOutlet | Swift Type | XCUIElement Type | AID Set? |
|---|---|---|---|
| `imageView` | UIImageView | XCUIElementTypeImage | ❌ Not set |
| `titleLabel` | UILabel | XCUIElementTypeStaticText | ❌ Not set |
| `descriptionLabel` | UILabel | XCUIElementTypeStaticText | ❌ Not set |

### 6.2 Locator Priority for This Screen

```
Priority 1 — Accessibility ID     ❌ NOT AVAILABLE — no accessibilityIdentifier set in source
Priority 2 — Class Chain          ✅ USE THIS — positional + nav bar context
Priority 3 — NSPredicate String   ✅ USE THIS — type + visible filter
Priority 4 — XPath                ⚠️  Last resort only
```

---

### 6.3 Priority 1 — Accessibility ID

```
❌ NOT AVAILABLE

No accessibilityIdentifier values are assigned in DetailsMovieViewController.swift.
The IBOutlets (imageView, titleLabel, descriptionLabel) have no .accessibilityIdentifier set.

→ Skip to Priority 2 (Class Chain) or Priority 3 (NSPredicate).

To unlock Priority 1, add to configView() in DetailsMovieViewController.swift:
    imageView.accessibilityIdentifier       = "details_poster"
    titleLabel.accessibilityIdentifier      = "details_title"
    descriptionLabel.accessibilityIdentifier = "details_description"
```

---

### 6.4 Priority 2 — Class Chain

```java
// Confirm screen — navigation bar named "Movie Details"
WebElement navBarCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeNavigationBar[`name == \"Movie Details\"`]")
);

// Poster image — first XCUIElementTypeImage on screen
WebElement posterCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeImage[1]")
);

// Title label — first static text
WebElement titleCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeStaticText[1]")
);

// Description label — second static text
WebElement descriptionCc = driver.findElement(
    AppiumBy.iOSClassChain("**/XCUIElementTypeStaticText[2]")
);

// Back button — first button inside the navigation bar
WebElement backCc = driver.findElement(
    AppiumBy.iOSClassChain(
        "**/XCUIElementTypeNavigationBar[`name == \"Movie Details\"`]/XCUIElementTypeButton[1]"
    )
);

// All static texts on screen (title + description together)
List<WebElement> allTextsCc = driver.findElements(
    AppiumBy.iOSClassChain("**/XCUIElementTypeStaticText")
);
```

---

### 6.5 Priority 3 — NSPredicate String

```java
// Confirm screen via navigation bar
WebElement navBarPred = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeNavigationBar\" AND name == \"Movie Details\""
    )
);

// Poster image — visible image on screen
WebElement posterPred = driver.findElement(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeImage\" AND visible == true")
);

// All visible static text labels
List<WebElement> allLabelsPred = driver.findElements(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeStaticText\" AND visible == true")
);

// Back button
WebElement backPred = driver.findElement(
    AppiumBy.iOSNsPredicateString("type == \"XCUIElementTypeButton\" AND visible == true")
);
```

---

### 6.6 Priority 4 — XPath (last resort)

```java
// Navigation bar
WebElement navBarXp = driver.findElement(
    By.xpath("//XCUIElementTypeNavigationBar[@name='Movie Details']")
);

// Back button inside nav bar
WebElement backXp = driver.findElement(
    By.xpath("//XCUIElementTypeNavigationBar//XCUIElementTypeButton[1]")
);

// First image (poster)
WebElement posterXp = driver.findElement(By.xpath("//XCUIElementTypeImage[1]"));

// First static text (title)
WebElement titleXp = driver.findElement(By.xpath("//XCUIElementTypeStaticText[1]"));

// Second static text (description)
WebElement descriptionXp = driver.findElement(By.xpath("//XCUIElementTypeStaticText[2]"));
```

---

## 7. Appium Capabilities Reference

```java
import io.appium.java_client.ios.IOSDriver;
import io.appium.java_client.ios.options.XCUITestOptions;
import java.net.URL;
import java.time.Duration;

XCUITestOptions options = new XCUITestOptions()
    .setPlatformVersion("18.0")                 // simulator OS version
    .setDeviceName("iPhone 17 Pro")
    .setUdid("0CEC38A6-4EAD-4D44-B8F4-DF6D8902D6FD")
    .setBundleId("com.okanorkun.MovieApp")
    .setNoReset(true)
    .setFullReset(false)
    .setIsSimulator(true)
    .setWdaLaunchTimeout(Duration.ofSeconds(120))
    .setNewCommandTimeout(Duration.ofSeconds(300));

// App path (use for installing/reinstalling)
options.setApp(
    "/Users/ibrahimnasr/Desktop/ios-automation/ios-movie-apps/Swift-MovieApp/MovieApp/" +
    "Build/Build/Products/Debug-iphonesimulator/MovieApp.app"
);

// Optional — set Xcode org ID for real device (not required for simulator)
// options.setXcodeOrgId("<your-team-id>");
// options.setXcodeSigningId("iPhone Developer");

IOSDriver driver = new IOSDriver(new URL("http://127.0.0.1:4723"), options);
```

---

## 8. XCUITest Native Reference

### 8.1 All Accessibility IDs in XCUITest

```swift
import XCTest

class FilmFoxUITests: XCTestCase {

    let app = XCUIApplication()

    override func setUpWithError() throws {
        continueAfterFailure = false
        app.launch()
    }

    // MARK: — Home Screen Static Elements

    func testHomeStaticElements() {
        let header       = app.otherElements["home_header"]
        let table        = app.tables["movies_table"]
        let carousel     = app.collectionViews["carousel_collection"]
        let sectionTabs  = app.scrollViews["section_tab_bar"]

        let btnSearch    = app.buttons["btn_search"]
        let btnTrophy    = app.buttons["btn_trophy"]
        let btnFilter    = app.buttons["btn_filter"]
        let btnSort      = app.buttons["btn_sort"]
        let btnSaved     = app.buttons["btn_saved"]

        let tabTrending  = app.buttons["tab_trending"]
        let tabRomantic  = app.buttons["tab_romantic"]
        let tabAction    = app.buttons["tab_action"]
        let tabTopRated  = app.buttons["tab_top_rated"]

        XCTAssertTrue(table.exists)
        XCTAssertTrue(btnSearch.exists)
    }

    // MARK: — Dynamic Movie Cell Elements

    func testMovieCell() {
        let card   = app.cells["movie_card_inception"]
        let play   = app.buttons["btn_play_inception"]
        let rating = app.otherElements["rating_badge_inception"]

        XCTAssertTrue(card.exists)
        play.tap()
    }

    // MARK: — Carousel Cell Elements

    func testCarouselCell() {
        let card   = app.cells["carousel_card_the_avengers"]
        let play   = app.buttons["carousel_play_the_avengers"]
        let title  = app.staticTexts["carousel_title_the_avengers"]
        let rating = app.staticTexts["carousel_rating_the_avengers"]

        XCTAssertTrue(card.exists)
    }

    // MARK: — Top Rated Screen

    func testTopRatedScreen() {
        app.buttons["btn_trophy"].tap()

        let collection     = app.collectionViews["top_rated_collection"]
        let pageIndicator  = app.pageIndicators["page_indicator"]
        let infoBanner     = app.staticTexts["top_rated_info_banner"]
        let btnBack        = app.buttons["btn_back"]

        // Dynamic cells (rank 1-indexed)
        let card1   = app.cells["top_rated_card_1"]
        let watch1  = app.buttons["btn_watch_now_1"]
        let title1  = app.staticTexts["top_rated_title_1"]
        let rating1 = app.staticTexts["top_rated_rating_1"]

        XCTAssertTrue(collection.exists)
        XCTAssertTrue(card1.exists)
    }

    // MARK: — Details Screen (fallback — no AID set yet)

    func testDetailsScreen() {
        // Tap first movie cell
        app.tables["movies_table"].cells.firstMatch.tap()

        let navBar      = app.navigationBars["Movie Details"]
        let poster      = app.images.firstMatch
        let titleLabel  = app.staticTexts.element(boundBy: 0)
        let description = app.staticTexts.element(boundBy: 1)

        XCTAssertTrue(navBar.exists)
        XCTAssertTrue(poster.exists)
    }
}
```

---

### 8.2 Class Chain in XCUITest (iOSNsPredicate / coordinate fallback)

XCUITest does not expose Class Chain natively — it is an Appium abstraction over the XCUI query engine. Use `XCUIElementQuery` with `.matching(identifier:)` or `NSPredicate` instead:

```swift
// NSPredicate in XCUITest — all play buttons
let playPredicate = NSPredicate(format: "identifier BEGINSWITH 'btn_play_'")
let allPlayButtons = app.buttons.matching(playPredicate)
XCTAssertGreaterThan(allPlayButtons.count, 0)

// NSPredicate — all carousel cards
let carouselPredicate = NSPredicate(format: "identifier BEGINSWITH 'carousel_card_'")
let carouselCards = app.cells.matching(carouselPredicate)

// NSPredicate — top rated cards
let topRatedPredicate = NSPredicate(format: "identifier BEGINSWITH 'top_rated_card_'")
let topRatedCards = app.cells.matching(topRatedPredicate)

// NSPredicate — all tab buttons
let tabPredicate = NSPredicate(format: "identifier BEGINSWITH 'tab_'")
let allTabs = app.buttons.matching(tabPredicate)
XCTAssertEqual(allTabs.count, 4)
```

---

## 9. Localization Support

### 9.1 Localization Strategy

| Attribute | Localized? | Safe as Locator? | Notes |
|---|---|---|---|
| `accessibilityIdentifier` | ❌ Never | ✅ Always safe | Set in Swift code, language-independent |
| `label` / visible text | ✅ Yes | ⚠️ Breaks on locale change | Avoid in primary locators |
| `value` | ✅ Sometimes | ⚠️ Fragile | Placeholder text may be localized |
| `name` (AID) | ❌ Never | ✅ Always safe | Same as `accessibilityIdentifier` |

**Rule:** Always use `accessibilityIdentifier` (Priority 1) as the primary locator. It never changes regardless of language. Only fall back to `label` / text when AID is unavailable — and parameterize those strings via a locale file.

---

### 9.2 Localization File Structure

```
localize/
├── en/
│   └── Localizable.strings          ← English UI strings
└── ar/
    └── Localizable.strings          ← Arabic UI strings
```

---

### 9.3 localize/en/Localizable.strings

```properties
/* MainViewController */
"main_tab_trending"       = "Trending";
"main_tab_romantic"       = "Romantic";
"main_tab_action"         = "Action";
"main_tab_top_rated"      = "Top Rated";
"main_btn_search"         = "Search";
"main_btn_filter"         = "Filter";
"main_btn_sort"           = "Sort";
"main_btn_saved"          = "Saved";
"main_btn_play"           = "Watch Now";

/* TopRatedViewController */
"top_rated_btn_back"      = "Back";
"top_rated_btn_watch_now" = "Watch Now";
"top_rated_info_banner"   = "Top Rated Movies of All Time";

/* DetailsMovieViewController */
"details_nav_title"       = "Movie Details";
"details_label_title"     = "Title";
"details_label_desc"      = "Description";
```

---

### 9.4 localize/ar/Localizable.strings

```properties
/* MainViewController */
"main_tab_trending"       = "رائج";
"main_tab_romantic"       = "رومانسي";
"main_tab_action"         = "أكشن";
"main_tab_top_rated"      = "الأعلى تقييماً";
"main_btn_search"         = "بحث";
"main_btn_filter"         = "تصفية";
"main_btn_sort"           = "ترتيب";
"main_btn_saved"          = "محفوظ";
"main_btn_play"           = "شاهد الآن";

/* TopRatedViewController */
"top_rated_btn_back"      = "رجوع";
"top_rated_btn_watch_now" = "شاهد الآن";
"top_rated_info_banner"   = "أفضل الأفلام على الإطلاق";

/* DetailsMovieViewController */
"details_nav_title"       = "تفاصيل الفيلم";
"details_label_title"     = "العنوان";
"details_label_desc"      = "الوصف";
```

---

### 9.5 Java Localization Enum

```java
import java.io.FileInputStream;
import java.util.Arrays;
import java.util.Properties;

public enum LocaleStrings {

    // — MainViewController —
    MAIN_TAB_TRENDING    ("MainViewController", "main_tab_trending"),
    MAIN_TAB_ROMANTIC    ("MainViewController", "main_tab_romantic"),
    MAIN_TAB_ACTION      ("MainViewController", "main_tab_action"),
    MAIN_TAB_TOP_RATED   ("MainViewController", "main_tab_top_rated"),
    MAIN_BTN_SEARCH      ("MainViewController", "main_btn_search"),
    MAIN_BTN_FILTER      ("MainViewController", "main_btn_filter"),
    MAIN_BTN_SORT        ("MainViewController", "main_btn_sort"),
    MAIN_BTN_SAVED       ("MainViewController", "main_btn_saved"),
    MAIN_BTN_PLAY        ("MainViewController", "main_btn_play"),

    // — TopRatedViewController —
    TOP_RATED_BTN_BACK      ("TopRatedViewController", "top_rated_btn_back"),
    TOP_RATED_BTN_WATCH_NOW ("TopRatedViewController", "top_rated_btn_watch_now"),
    TOP_RATED_INFO_BANNER   ("TopRatedViewController", "top_rated_info_banner"),

    // — DetailsMovieViewController —
    DETAILS_NAV_TITLE    ("DetailsMovieViewController", "details_nav_title"),
    DETAILS_LABEL_TITLE  ("DetailsMovieViewController", "details_label_title"),
    DETAILS_LABEL_DESC   ("DetailsMovieViewController", "details_label_desc");

    public  final String screen;
    private final String key;
    private static final Properties strings = new Properties();

    LocaleStrings(String screen, String key) {
        this.screen = screen;
        this.key    = key;
    }

    /** Load locale file — call once before tests. locale: "en" or "ar" */
    public static void load(String locale) throws Exception {
        String path = "localize/" + locale + "/Localizable.strings";
        strings.load(new FileInputStream(path));
    }

    /** Resolved string value for the currently loaded locale */
    public String value() {
        return strings.getProperty(key, key); // fallback to key name if not loaded
    }

    /** All constants that belong to a given controller */
    public static LocaleStrings[] forScreen(String controllerName) {
        return Arrays.stream(values())
                     .filter(e -> e.screen.equals(controllerName))
                     .toArray(LocaleStrings[]::new);
    }
}
```

---

### 9.6 Using Localized Strings in Locators

#### When AID is set — locale does NOT matter
```java
// ✅ Safe in any language — accessibilityIdentifier never changes
WebElement btnSearch   = driver.findElement(AppiumBy.accessibilityId("btn_search"));
WebElement moviesTable = driver.findElement(AppiumBy.accessibilityId("movies_table"));
WebElement card1       = driver.findElement(AppiumBy.accessibilityId("top_rated_card_1"));
```

#### When AID is NOT set — use enum constant via NSPredicate
```java
// Load locale once at test setup
LocaleStrings.load("en"); // or "ar"

// DetailsMovieViewController — no AID, fall back to localized nav bar title
WebElement navBar = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeNavigationBar\" AND name == \"" + LocaleStrings.DETAILS_NAV_TITLE.value() + "\""
    )
);

// TopRatedViewController — Watch Now button by visible label (fallback when no AID)
WebElement watchNowBtn = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeButton\" AND label == \"" + LocaleStrings.TOP_RATED_BTN_WATCH_NOW.value() + "\""
    )
);

// DetailsMovieViewController — Class Chain with localized nav bar title
WebElement navBarCc = driver.findElement(
    AppiumBy.iOSClassChain(
        "**/XCUIElementTypeNavigationBar[`name == \"" + LocaleStrings.DETAILS_NAV_TITLE.value() + "\"`]"
    )
);

// MainViewController — tab label query via enum
WebElement trendingTab = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeButton\" AND label == \"" + LocaleStrings.MAIN_TAB_TRENDING.value() + "\""
    )
);
```

#### English vs Arabic — same test, different locale
```java
// English run
LocaleStrings.load("en");
WebElement backBtnEn = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeButton\" AND label == \"" + LocaleStrings.TOP_RATED_BTN_BACK.value() + "\""
    )
);
// Resolves to: label == "Back"

// Arabic run (RTL)
LocaleStrings.load("ar");
WebElement backBtnAr = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeButton\" AND label == \"" + LocaleStrings.TOP_RATED_BTN_BACK.value() + "\""
    )
);
// Resolves to: label == "رجوع"
```

#### NSPredicate CONTAINS — partial match (safer for RTL)
```java
// Use CONTAINS instead of == when text may have RTL markers or extra whitespace
WebElement backBtnRtl = driver.findElement(
    AppiumBy.iOSNsPredicateString(
        "type == \"XCUIElementTypeButton\" AND label CONTAINS \"" + LocaleStrings.TOP_RATED_BTN_BACK.value() + "\""
    )
);
```

#### Load all strings for one controller at once
```java
// Print all keys that belong to MainViewController
for (LocaleStrings s : LocaleStrings.forScreen("MainViewController")) {
    System.out.println(s.name() + " = " + s.value());
}
// Output (en):
// MAIN_TAB_TRENDING = Trending
// MAIN_BTN_SEARCH   = Search
// ...
```

---

### 9.7 Localization — Which Locators Are Affected

| Screen | Element | AID Set? | Locale Impact |
|---|---|---|---|
| Home | `btn_search` | ✅ | None — use AID |
| Home | `btn_trophy` | ✅ | None — use AID |
| Home | `tab_trending` | ✅ | None — use AID |
| Home | `tab_romantic` | ✅ | None — use AID |
| Home | `tab_action` | ✅ | None — use AID |
| Home | `tab_top_rated` | ✅ | None — use AI# iOS Locator Extraction Master Prompt

You are an elite iOS reverse-engineering and automation analysis engine.

Your task is to analyze an iOS repository and extract:
- all user-facing controllers/screens
- UI hierarchy
- accessibility locators
- dynamic locator rules
- reusable UI components
- navigation flows
- automation stability metadata

The final result will be used to:
- build Appium frameworks
- generate locator repositories
- generate AI automation
- build XCUITest mapping
- generate automation documentation
- create scalable locator systems

--------------------------------------------------
PRIMARY GOAL
--------------------------------------------------

Extract ONLY:
- UIViewControllers
- SwiftUI Views
- Storyboards
- XIB screens
- reusable UI components
- collection/table cells
- navigation flows
- automation locators

Ignore:
- networking
- API services
- repositories
- models
- analytics
- helpers
- utilities
- tests
- dependency injection
- storage
- business logic
- build configs

--------------------------------------------------
STEP 1 — DETECT UI ARCHITECTURE
--------------------------------------------------

Detect:
- UIKit
- SwiftUI
- Storyboards
- XIB
- Programmatic UI
- Hybrid architecture

Identify:
- NavigationStack
- NavigationView
- UIViewController
- UITableViewController
- UICollectionViewController
- UIHostingController
- Storyboard references
- XIB loading
- Tab bars
- Split views
- modal presentations

--------------------------------------------------
STEP 2 — DETECT ALL SCREENS
--------------------------------------------------

Find ALL user-facing screens.

SwiftUI patterns:
- struct *: View
- body: some View
- NavigationStack
- TabView
- sheet(
- fullScreenCover(

UIKit patterns:
- UIViewController
- UITableViewController
- UICollectionViewController
- viewDidLoad()
- loadView()
- present(
- pushViewController(

Storyboard/XIB:
- .storyboard
- .xib
- IBOutlet
- IBAction

Folders:
- Views
- Screens
- Features
- Modules
- UI
- Presentation
- Components
- Scene
- Pages

--------------------------------------------------
STEP 3 — SCREEN METADATA
--------------------------------------------------

For each controller return:
- controller_name
- screen_name
- file_path
- framework
- presentation_type
- parent_controller
- child_controllers
- navigation_entry
- navigation_exit
- screen_purpose
- load_trigger
- visible_components
- reusable_components
- dynamic_content_presence

--------------------------------------------------
STEP 4 — EXTRACT FULL UI HIERARCHY
--------------------------------------------------

Extract ALL visible elements:
- buttons
- labels
- images
- textfields
- search bars
- toggles
- sliders
- switches
- cards
- table cells
- collection cells
- navigation bars
- tab bars
- alerts
- sheets
- modals
- banners
- loaders
- progress views

For EACH element extract:
- element_name
- swift_type
- xcui_element_type
- parent_container
- visible_text
- placeholder
- accessibility_identifier
- accessibility_label
- accessibility_value
- localization_key
- gesture_support
- tap_action
- long_press
- swipe_support
- scroll_context
- conditional_rendering
- animation_presence

--------------------------------------------------
STEP 5 — LOCATOR STRATEGY RULES
--------------------------------------------------

Locator priority:
1. accessibilityIdentifier
2. iOS Class Chain
3. NSPredicate
4. XPath only if unavoidable

NEVER prefer XPath when:
- Class Chain is possible
- accessibilityIdentifier exists
- parent-child relationship exists

Prefer Class Chain when:
- no accessibilityIdentifier exists
- reusable cell exists
- dynamic list exists
- nested hierarchy exists
- stable parent exists
- UICollectionView exists
- UITableView exists

Prefer NSPredicate when:
- filtering by type
- filtering by visibility
- partial matching
- dynamic identifiers
- localization fallback

Use XPath ONLY:
- when all other strategies fail
- when hierarchy is unknown
- when native hierarchy is unstable

--------------------------------------------------
STEP 6 — GENERATE LOCATORS
--------------------------------------------------

For EACH element generate:
- best_locator_strategy
- best_locator
- accessibility_locator
- class_chain_locator
- nspredicate_locator
- xpath_locator
- fallback_locator
- locator_confidence_score
- localization_safe
- automation_stability

--------------------------------------------------
STEP 7 — DYNAMIC LOCATOR DETECTION
--------------------------------------------------

Detect patterns:
- btn_{name}
- card_{id}
- movie_{safeID}
- row_{index}
- item_{slug}
- title_{rank}
- dynamic_uuid
- generated identifiers

Detect normalization logic:
- lowercased()
- replacingOccurrences
- slugify
- remove spaces
- remove apostrophes
- remove symbols
- rank-based
- index-based

Extract:
- dynamic_pattern
- normalization_steps
- example_input
- example_output
- recommended_strategy
- dynamic_locator_confidence

--------------------------------------------------
STEP 8 — REUSABLE COMPONENT DETECTION
--------------------------------------------------

Find:
- UITableViewCell
- UICollectionViewCell
- reusable SwiftUI components
- headers
- footers
- cards
- custom controls

For each reusable component:
- component_name
- component_type
- file_path
- used_in_controllers
- reuse_identifier
- nested_components
- locator_strategy
- dynamic_patterns
- automation_risk

--------------------------------------------------
STEP 9 — ACCESSIBILITY AUDIT
--------------------------------------------------

Detect:
- missing accessibilityIdentifier
- duplicate identifiers
- weak labels
- inaccessible buttons
- image-only controls
- localization risks
- RTL risks
- unstable dynamic IDs
- text-based locator risks

Generate:
- recommended accessibilityIdentifier
- automation-safe naming
- accessibility fixes

--------------------------------------------------
STEP 10 — AUTOMATION SCORE
--------------------------------------------------

Generate automation score for EACH controller.

Scoring:
- accessibility coverage: 30
- locator stability: 25
- navigation clarity: 15
- dynamic content stability: 15
- localization safety: 10
- animation/scroll stability: 5

Ratings:
- 90-100 = Excellent
- 75-89 = Good
- 60-74 = Medium
- 40-59 = Weak
- 0-39 = Poor

Return:
- total_score
- rating
- weaknesses
- risks
- improvement_recommendations

--------------------------------------------------
STEP 11 — NAVIGATION GRAPH
--------------------------------------------------

Generate navigation graph:
- push navigation
- modal navigation
- sheet navigation
- tab navigation
- deep links
- root controller
- authentication gates

--------------------------------------------------
STEP 12 — SCROLLABLE STRUCTURES
--------------------------------------------------

Detect:
- UITableView
- UICollectionView
- ScrollView
- LazyVStack
- LazyHStack
- carousels
- pagers

Extract:
- scroll strategy
- pagination support
- infinite scrolling
- visible cell strategy
- reusable cell risks

--------------------------------------------------
STEP 13 — LOCALIZATION IMPACT
--------------------------------------------------

Detect:
- NSLocalizedString
- Localizable.strings
- RTL support
- locale-based labels

Classify locators:
- locale safe
- locale unsafe

Recommend:
- accessibilityIdentifier fallback
- localization abstraction

--------------------------------------------------
STEP 14 — OUTPUT RULES
--------------------------------------------------

Return VALID JSON ONLY.

NO markdown.
NO explanations.
NO test code.
NO Appium code.
NO Page Object generation.
NO prose outside JSON.

--------------------------------------------------
REQUIRED JSON STRUCTURE
--------------------------------------------------

{
  "app_summary": {
    "app_name": "",
    "bundle_id": "",
    "ui_architecture": "",
    "frameworks_detected": [],
    "storyboard_usage": "",
    "xib_usage": "",
    "automation_readiness_score": 0,
    "automation_readiness_rating": ""
  },
  "locator_strategy_rules": {
    "priority_order": [
      "accessibility_id",
      "ios_class_chain",
      "nspredicate",
      "xpath"
    ],
    "preferred_strategy": "accessibility_id",
    "fallback_strategy": "ios_class_chain",
    "avoid": "xpath"
  },
  "controllers": [
    {
      "controller_name": "",
      "screen_name": "",
      "file_path": "",
      "framework": "",
      "presentation_type": "",
      "parent_controller": "",
      "child_controllers": [],
      "screen_purpose": "",
      "screen_load_locator": {
        "strategy": "",
        "locator": "",
        "confidence_score": 0
      },
      "automation_score": {
        "total": 0,
        "rating": "",
        "breakdown": {
          "accessibility_coverage": 0,
          "locator_stability": 0,
          "navigation_clarity": 0,
          "dynamic_content_stability": 0,
          "localization_safety": 0,
          "scroll_animation_stability": 0
        }
      },
      "elements": [
        {
          "element_name": "",
          "swift_type": "",
          "xcui_element_type": "",
          "parent_container": "",
          "visible_text": "",
          "accessibility_identifier": "",
          "accessibility_label": "",
          "best_locator_strategy": "",
          "best_locator": "",
          "class_chain_locator": "",
          "nspredicate_locator": "",
          "xpath_locator": "",
          "locator_confidence_score": 0,
          "is_dynamic": false,
          "dynamic_pattern": "",
          "action": "",
          "automation_note": ""
        }
      ],
      "dynamic_locator_patterns": [],
      "reusable_components": [],
      "navigation_actions": [],
      "risks": [],
      "recommended_accessibility_fixes": []
    }
  ],
  "global_dynamic_rules": [],
  "global_locator_risks": [],
  "final_recommendations": []
}

--------------------------------------------------
VERY IMPORTANT RULES
--------------------------------------------------

- Prefer accessibilityIdentifier always
- Prefer Class Chain over XPath
- Use XPath only as last resort
- Detect reusable cells
- Detect dynamic identifiers
- Detect nested collection structures
- Detect UITableView/UICollectionView hierarchy
- Detect modal transitions
- Detect runtime-generated UI
- Detect hidden automation risks
- Detect unstable text-based locators
- Detect localization risks
- Detect animation risks
- Detect scroll instability
- Detect reused identifiers
- Every controller must be mapped
- Every visible screen must be extracted
- Every locator must include confidence score
- Every screen must include automation score
```D |
| Home | `movies_table` | ✅ | None — use AID |
| Home | `movie_card_{safeID}` | ✅ | None — safeID derived from data, not UI label |
| Home | `btn_play_{safeID}` | ✅ | None — use AID |
| TopRated | `btn_back` | ✅ | None — use AID |
| TopRated | `btn_watch_now_{rank}` | ✅ | None — use AID |
| TopRated | `top_rated_info_banner` | ✅ | None — use AID |
| Details | Navigation bar title | ❌ | ⚠️ Localized — use `LocaleStrings.get("nav_movie_details")` |
| Details | `imageView` | ❌ | None — images are not localized |
| Details | `titleLabel` | ❌ | ⚠️ Content changes per locale |
| Details | `descriptionLabel` | ❌ | ⚠️ Content changes per locale |
