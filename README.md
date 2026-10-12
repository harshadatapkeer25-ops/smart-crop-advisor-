# smart-crop-advisor-


import streamlit as st
import pandas as pd

# --------------------------------------------------
# PAGE CONFIGURATION
# --------------------------------------------------
st.set_page_config(
    page_title="Smart Crop Advisor",
    page_icon="🌱",
    layout="wide"
)


# --------------------------------------------------
# CROP RECOMMENDATION ENGINE
# --------------------------------------------------
class CropRecommendationEngine:

    def __init__(self):
        self.crop_database = [
            {
                "crop": "Soybean",
                "soil_types": ["Black Soil", "Alluvial Soil"],
                "water_req": "Medium",
                "season": "Kharif",
                "avg_price_per_kg": 48.0,
                "cost_per_acre": 14000,
                "expected_yield_kg": 800,
                "harvest_days": 95
            },
            {
                "crop": "Maize",
                "soil_types": [
                    "Black Soil", "Red Soil", "Loamy Soil"
                ],
                "water_req": "Medium",
                "season": "Kharif",
                "avg_price_per_kg": 22.0,
                "cost_per_acre": 16000,
                "expected_yield_kg": 1800,
                "harvest_days": 100
            },
            {
                "crop": "Tur (Pigeon Pea)",
                "soil_types": ["Black Soil", "Red Soil"],
                "water_req": "Low",
                "season": "Kharif",
                "avg_price_per_kg": 70.0,
                "cost_per_acre": 12000,
                "expected_yield_kg": 500,
                "harvest_days": 160
            },
            {
                "crop": "Wheat",
                "soil_types": ["Alluvial Soil", "Black Soil"],
                "water_req": "High",
                "season": "Rabi",
                "avg_price_per_kg": 26.0,
                "cost_per_acre": 15000,
                "expected_yield_kg": 1500,
                "harvest_days": 120
            }
        ]

    def recommend_top_crops(
        self, soil_type, water_availability, season
    ):
        results = []

        for crop in self.crop_database:
            score = 0

            # Soil matching
            if soil_type in crop["soil_types"]:
                score += 40
            else:
                score += 10

            # Water matching
            if crop["water_req"] == water_availability:
                score += 30
            elif water_availability == "High":
                score += 20
            else:
                score += 5

            # Season matching
            if crop["season"] == season:
                score += 30

            # Financial calculations
            revenue = (
                crop["avg_price_per_kg"]
                * crop["expected_yield_kg"]
            )

            profit = revenue - crop["cost_per_acre"]

            # Suitability rating
            if score >= 90:
                stars = "★★★★★"
            elif score >= 70:
                stars = "★★★★☆"
            else:
                stars = "★★★☆☆"

            results.append({
                "Crop": crop["crop"],
                "Suitability Score": score,
                "Rating": stars,
                "Expected Price": (
                    f"₹{crop['avg_price_per_kg'] - 2:.0f}"
                    f"–₹{crop['avg_price_per_kg'] + 3:.0f}/kg"
                ),
                "Cost per Acre (₹)": crop["cost_per_acre"],
                "Expected Revenue (₹)": revenue,
                "Estimated Profit (₹)": profit,
                "Harvest Period (Days)": crop["harvest_days"]
            })

        # Sort numerically by score and then profit
        results.sort(
            key=lambda item: (
                item["Suitability Score"],
                item["Estimated Profit (₹)"]
            ),
            reverse=True
        )

        return results[:3]


# --------------------------------------------------
# INITIALIZE APP
# --------------------------------------------------
engine = CropRecommendationEngine()


# --------------------------------------------------
# APP HEADER
# --------------------------------------------------
st.title("🌱 Smart Crop Advisor")
st.subheader("Find suitable crops and estimate your farm profit")

st.write(
    "Select your soil type, water availability, and farming "
    "season to get your top 3 crop recommendations."
)

st.divider()


# --------------------------------------------------
# INPUT SECTION
# --------------------------------------------------
st.header("🚜 Enter Your Farm Details")

col1, col2, col3 = st.columns(3)

with col1:
    soil_type = st.selectbox(
        "Select Soil Type",
        [
            "Black Soil",
            "Red Soil",
            "Alluvial Soil",
            "Loamy Soil"
        ]
    )

with col2:
    water_availability = st.selectbox(
        "Water Availability",
        ["Low", "Medium", "High"]
    )

with col3:
    season = st.selectbox(
        "Select Season",
        ["Kharif", "Rabi"]
    )

recommend_button = st.button(
    "🌾 Recommend Crops",
    type="primary",
    use_container_width=True
)


# --------------------------------------------------
# RECOMMENDATION RESULTS
# --------------------------------------------------
if recommend_button:

    recommendations = engine.recommend_top_crops(
        soil_type=soil_type,
        water_availability=water_availability,
        season=season
    )

    st.divider()
    st.header("🌿 Top 3 Recommended Crops")

    st.caption(
        f"Soil: {soil_type} | "
        f"Water: {water_availability} | "
        f"Season: {season}"
    )

    # Display crop cards
    cols = st.columns(3)

    for index, crop in enumerate(recommendations):
        with cols[index]:
            st.subheader(f"{index + 1}. {crop['Crop']}")

            st.write(crop["Rating"])

            st.metric(
                "Suitability Score",
                f"{crop['Suitability Score']}%"
            )

            st.metric(
                "Estimated Profit / Acre",
                f"₹{crop['Estimated Profit (₹)']:,.0f}"
            )

            st.write(
                f"**Expected price:** {crop['Expected Price']}"
            )

            st.write(
                f"**Cost per acre:** "
                f"₹{crop['Cost per Acre (₹)']:,.0f}"
            )

            st.write(
                f"**Expected revenue:** "
                f"₹{crop['Expected Revenue (₹)']:,.0f}"
            )

            st.write(
                f"**Harvest period:** "
                f"{crop['Harvest Period (Days)']} days"
            )

    # Comparison table
    st.divider()
    st.subheader("📊 Crop Comparison")

    df = pd.DataFrame(recommendations)

    df["Suitability Score"] = df[
        "Suitability Score"
    ].astype(str) + "%"

    for column in [
        "Cost per Acre (₹)",
        "Expected Revenue (₹)",
        "Estimated Profit (₹)"
    ]:
        df[column] = df[column].map(
            lambda value: f"₹{value:,.0f}"
        )

    st.dataframe(
        df,
        use_container_width=True,
        hide_index=True
    )

    # Download recommendations
    csv_data = pd.DataFrame(recommendations).to_csv(
        index=False
    ).encode("utf-8")

    st.download_button(
        label="⬇️ Download Recommendations (CSV)",
        data=csv_data,
        file_name="crop_recommendations.csv",
        mime="text/csv"
    )


# --------------------------------------------------
# FOOTER
# --------------------------------------------------
st.divider()

st.subheader("ℹ️ About This App")

st.write(
    "Smart Crop Advisor recommends crops using a simple "
    "soil, water, and season scoring system."
)

st.warning(
    "Important: Crop prices, yields, and cultivation costs "
    "are illustrative sample values, not live market data. "
    "Actual results depend on location, weather, soil health, "
    "seed variety, input costs, and market prices. Verify local "
    "agricultural recommendations before making farming decisions."
)

st.caption("Smart Crop Advisor | Educational Project")
