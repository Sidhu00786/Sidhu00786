async async function fetchAIPoweredAd(productId) {
  try {
    const adRes = await fetch(`http://localhost:4000/api/generate-ai-ad?productId=${productId}`, {
      headers: { 'Content-Type': 'application/json' }
    });

    // Log raw response for debugging
    console.log(`AI Ad API response for product ${productId}:`, adRes);

    if (!adRes.ok) {
      const errorBody = await adRes.text();
      console.error(`API Error ${adRes.status} for product ${productId}:`, errorBody);
      throw new Error(`AI ad generation failed: ${adRes.statusText}`);
    }

    const adData = await adRes.json();
    console.log(`Parsed AI Ad data for ${productId}:`, adData);

    // Validate response structure
    if (!adData?.ad?.text || !adData?.ad?.keywords) {
      console.error('Invalid AI ad structure for product:', productId, adData);
      throw new Error('Invalid AI ad response structure');
    }

    return {
      aiAdText: adData.ad.text,
      keywords: adData.ad.keywords,
      generatedAt: new Date().toISOString()
    };

  } catch (err) {
    console.error(`AI Ad generation failed for ${productId}:`, err);
    // Fallback ad content
    return {
      aiAdText: "✨ Special Limited Offer! Get this amazing product while supplies last!",
      keywords: ["special", "offer", "limited"],
      isFallback: true
    };
  }
}

// In your main fetchProducts function:
async function fetchProducts() {
  try {
    const response = await fetch('http://localhost:4000/api/atomenick-products');
    
    if (!response.ok) {
      const errorText = await response.text();
      throw new Error(`Product fetch failed: ${response.status} - ${errorText}`);
    }

    const products = await response.json();
    
    // Add AI ads with error handling per product
    const productsWithAds = await Promise.all(
      products.map(async (product) => {
        try {
          const aiAd = await fetchAIPoweredAd(product.id);
          return { ...product, aiAd };
        } catch (err) {
          console.error(`Failed to get AI ad for ${product.id}:`, err);
          return {
            ...product,
            aiAd: {
              aiAdText: "🌟 Exclusive Deal! Don't miss this incredible opportunity!",
              keywords: ["exclusive", "deal", "opportunity"],
              isFallback: true
            }
          };
        }
      })
    );

    return productsWithAds;

  } catch (err) {
    console.error('Critical error fetching products:', err);
    throw new Error(`Failed to load products: ${err.message}`);
  }
} fetchAndGenerateAd() {
  try {
    const productRes = await fetch("http://localhost:4000/api/atomenick-products");
    if (!productRes.ok) {
      throw new Error(`Failed to fetch products: ${productRes.status} ${productRes.statusText}`);
    }

    const products = await productRes.json();

    const adRes = await fetch("http://localhost:4000/api/generate-ai-ad", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ products }),
    });

    if (!adRes.ok) {
      throw new Error(`AI ad generation failed: ${adRes.status} ${adRes.statusText}`);
    }

    const adData = await adRes.json();

    if (!adData || !adData.adText) {
      throw new Error("Malformed AI ad response: missing 'adText'");
    }

    displayAd(adData.adText);
  } catch (err) {
    console.error("Error fetching or generating AI ads:", err);

    // Optional: fallback ad
    const fallbackAd = "Explore our latest affordable collection today! Great deals await.";
    displayAd(fallbackAd);

    setErrorMessage(err.message || "Unknown error occurred while fetching or generating ads.");
  }
}
