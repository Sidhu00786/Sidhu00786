async function fetchAndGenerateAd() {
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
