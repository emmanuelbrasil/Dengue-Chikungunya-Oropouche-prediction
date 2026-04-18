# Dengue, Chikungunya and Oropouche Prediction Models

This repository contains predictive models (Random Forest using the `ranger` package) developed to support differential diagnosis between Dengue, Chikungunya, and Oropouche.

- **Interactive App:** [INDWELL Live Tool](https://pedrobrasil.shinyapps.io/INDWELL/)
- **Data:** Imputed clinical data available in `Imputed_data_2026-03-19.parquet`.

---

## 📂 Repository Structure

| File | Description |
| :--- | :--- |
| `Dengue_rf_reg_models_light.RData` | Trained model and Platt calibration for Dengue. |
| `Chikungunya_rf_reg_models_light.RData` | Trained model and Platt calibration for Chikungunya. |
| `Oropouche_rf_reg_models_light.RData` | Trained model and Platt calibration for Oropouche. |
| `Imputed_data_...parquet` | Anonymized dataset used for model development. |
| `All_analysis_scripts.zip` | Full analysis pipeline scripts. |

---

## 📋 Variable Dictionary (Model Inputs)

To use the models, you must provide a dataframe with the following predictors. All binary symptoms are factors with levels `c("No", "Yes")`.

### Clinical, Demographic, Signs and Symptoms (Predictors)
| Variable Code | Clinical Description |
| :--- | :--- | 
| `IDADE_ANOS_DT_NOTIFIC` | Age at notification |
| `ETNIA` | Indigenous ethnicity |
| `PCD` | Person with special needs |
| `MORADOR_RUA` | Homeless |
| `ESCOLARIDADE` | Education |
| `ZONA` | Home address |
| `DC_FEBRE` | Fever |
| `DC_CEFALEIA` | Headache |
| `DC_VOMITO` | Vomiting |
| `DC_DOR_COSTAS` | Back pain |
| `DC_ARTRITE` | Arthritis |
| `DC_PETEQUIAS` | Petechiae |
| `DC_PROVA_LACO` | Tourniquet test |
| `DC_MIALGIA` | Myalgia |
| `DC_EXANTEMA` | Rash |
| `DC_NAUSEAS` | Nausea |
| `DC_CONJUTIVITE` | Conjunctivitis |
| `DC_ARTRALGIA_INTENSA` | Intense arthralgia |
| `DC_LEUCOPENIA` | Leukopenia |
| `DC_DOR_RETROORBITAL` | Retro-orbital pain |
| `DC_DIABETES` | Diabetes |
| `DC_HEPATOPATIAS` | Hepatopathies |
| `DC_HIPER_ARTERIAL` | Arterial hypertension |
| `DC_AUTO_IMUNES` | Autoimmune diseases |
| `DC_HEMATOLOGICAS` | Hematological diseases |
| `DC_RENAL_CRONICA` | Chronic kidney disease |
| `DC_ACIDO_PEPTICA` | Peptic acid disease |
| `SEMANA_NUM` | First symptom week |
| `DENGUE` | Dengue diagnosis |
| `CHIKUNGUNYA` | Chikungunya diagnosis |
| `ZIKA` | Zika diagnosis |
| `OROPOUCHE` | Oropopuche diagnosis |
| `DENGUE_01` | Dengue diagnosis |
| `CHIKUNGUNYA_01` | Chikungunya diagnosis |
| `ZIKA_01` | Zika diagnosis |
| `OROPOUCHE_01` | Oropouche diagnosis |
| `C_CRIT_CONF_DESC` | Final Dengue classification criteria |
| `C_EVOLUC_CASO` | Case outcome |
| `C_AUTOCTONE` | Autochthonous case |
| `VALIDATION_1` | Validation temporal split 1 |
| `VALIDATION_2` | Validation temporal split 2 |
| `VALIDATION_3` | Validation random split |
---

## 💻 How to Run Predictions

```r
library(ranger)
## 1. Load your desired model
load("Dengue_rf_reg_models_light.RData")
load("Chikungunya_rf_reg_models_light.RData")
load("Oropouche_rf_reg_models_light.RData")

## 2. Create a template for a new patient
# Note: Ensure all symptoms are factors with levels c("No", "Yes")
# Note: The different models include different sets of predictors, and "newdata" includes all of them. 
newdata <- data.frame(
    # Demographic and Temporal
    SEMANA_NUM = 12,
    IDADE_ANOS_DT_NOTIFIC = 35,
    SEXO = factor("Female", levels = c("Female", "Male")),
    GESTANTE = factor("No", levels = c("No", "Yes")),
    RACA_COR = factor("White", levels = c("White", "Black", "Yellow", "Indigenous")),
    ESCOLARIDADE = factor("High School Complete", levels = c("Illiterate", 
                                                              "Elementary School Complete", 
                                                              "Middle School Complete", 
                                                              "High School Complete", 
                                                              "Higher Education Complete")),
    ZONA = factor("Urban", levels = c("Urban", "Rural", "Peri-urban")),
    
    # Clinical Predictors (Factors: No/Yes)
    DC_FEBRE = factor("Yes", levels = c("No", "Yes")),
    DC_CEFALEIA = factor("Yes", levels = c("No", "Yes")),
    DC_VOMITO = factor("No", levels = c("No", "Yes")),
    DC_DOR_COSTAS = factor("No", levels = c("No", "Yes")),
    DC_ARTRITE = factor("No", levels = c("No", "Yes")),
    DC_PETEQUIAS = factor("No", levels = c("No", "Yes")),
    DC_PROVA_LACO = factor("No", levels = c("No", "Yes")),
    DC_MIALGIA = factor("Yes", levels = c("No", "Yes")),
    DC_EXANTEMA = factor("No", levels = c("No", "Yes")),
    DC_NAUSEAS = factor("No", levels = c("No", "Yes")),
    DC_CONJUTIVITE = factor("No", levels = c("No", "Yes")),
    DC_ARTRALGIA_INTENSA = factor("No", levels = c("No", "Yes")),
    DC_LEUCOPENIA = factor("No", levels = c("No", "Yes")),
    DC_DOR_RETROORBITAL = factor("Yes", levels = c("No", "Yes")),
    
    # Comorbidities (Factors: No/Yes)
    DC_DIABETES = factor("No", levels = c("No", "Yes")),
    DC_HEPATOPATIAS = factor("No", levels = c("No", "Yes")),
    DC_HIPER_ARTERIAL = factor("No", levels = c("No", "Yes")),
    DC_AUTO_IMUNES = factor("No", levels = c("No", "Yes")),
    DC_HEMATOLOGICAS = factor("No", levels = c("No", "Yes")),
    DC_RENAL_CRONICA = factor("No", levels = c("No", "Yes")),
    DC_ACIDO_PEPTICA = factor("No", levels = c("No", "Yes"))
)

## 3. Get scaled probability
# Duengue probabilities
dengue_prob <- predict(dengue.reg.model, data = case)$predictions
dengue_plat_prob <- predict(dengue.reg.plat.model,
                            newdata = data.frame(prob = dengue_prob),
                            type = "response")
print(dengue_plat_prob)

# Chikungunya probabilities
chikungunya_prob <- predict(chikungunya.reg.model, data = newdata)$predictions
chikungunya_plat_prob <- predict(chikungunya.reg.plat.model, 
                      newdata = data.frame(prob = chikungunya_prob), 
                      type = "response")
print(chikungunya_plat_prob)

# Oropouche probabilities
oropouche_prob <- predict(oropouche.reg.model, data = newdata)$predictions
oropouche_plat_prob <- predict(oropouche.reg.plat.model, 
                       newdata = data.frame(prob = oro_raw), 
                       type = "response")
print(oropouche_plat_prob)

## 4. Generate a TG-ROC plot to explore inconclusive limits
# Example with dengue only, but can easily be adapted to chikungunya and oropouche.
# Note: Ensure inc.limits function is defined in your environment
# Copy the function below and paste it into your console
# Start copy here ----
inc.limits <- function(x, Inconclusive = 0.95) {
  condition <- which.min(abs(Inconclusive - x$table$Sensitivity))
  if (length(condition) > 1) {
    warn <- x$table$test.values[condition]
    warning(paste0("Sensitivity matches the minimum required at the following test values: ",
                   toString(warn), ". Highest one was chosen!"))
  }
  Se.pos <- condition[length(condition)]
  condition <- which.min(abs(Inconclusive - x$table$Specificity))
  if (length(condition) > 1) {
    warn <- x$table$test.values[condition]
    warning(paste0("Specificity matches the minimum required at the following\n                   test values: ",
                   toString(warn), ". First one was chosen!"))
  }
  Sp.pos <- condition[1]
  inc.output <- rbind(x$table[Se.pos, c("test.values", "Sensitivity",
                                    "Se.inf.cl", "Se.sup.cl", "Specificity", "Sp.inf.cl",
                                    "Sp.sup.cl", "PLR", "PLR.inf.cl", "PLR.sup.cl")],
                  x$table[Sp.pos,c("test.values", "Sensitivity", "Se.inf.cl", "Se.sup.cl",
                                   "Specificity", "Sp.inf.cl", "Sp.sup.cl",
                                   "PLR", "PLR.inf.cl","PLR.sup.cl")])
  rownames(inc.output) <- c("Lower inconclusive", "Upper inconclusive")
  inc.output
}
# Stop copying here ----

# 'dengue.reg.test.tgroc' is available in the loaded .RData
z_den <- inc.limits(dengue.reg.test.tgroc, Inconclusive = 0.90)

# Generate the Plot
tgroc_plot <- ggplot(dengue.reg.test.tgroc$table %>%
                       select(test.values, Sensitivity, Specificity),
                     aes(x = test.values)) +
    # Inconclusive Zone
    annotate("rect", 
             xmin = z_den$test.values[1], xmax = z_den$test.values[2], 
             ymin = 0, ymax = 1, fill = "gray", alpha = 0.3) +
    # Se & Sp Lines
    geom_line(aes(y = Sensitivity, colour = "Sensitivity"), linewidth = 1) +
    geom_line(aes(y = Specificity, colour = "Specificity"), linewidth = 1) +
    # Labels and Captions
    labs(
      title = "Dengue: TG-ROC Analysis",
      x = "Model Predictions (Calibrated Probability)",
      y = "Sensitivity & Specificity",
      caption = paste("Non-parametric inconclusive limits:",
                      sprintf("%.3f", z_den$test.values[1]), "-",
                      sprintf("%.3f", z_den$test.values[2]))
    ) +
    scale_color_manual(values = c("Sensitivity" = "blue", "Specificity" = "red")) +
    theme_light() +
    theme(
      legend.position = "top",
      legend.title = element_blank(),
      axis.text.x = element_text(angle = 30, hjust = 1, size = 8),
      plot.caption = element_text(hjust = 0.5, size = 9, face = "italic"),
      panel.grid.major = element_blank(),
      panel.grid.minor = element_blank()
    )

# View or Save
print(tgroc_plot)
# ggsave("TGROC_Dengue.png", width = 8, height = 6, dpi = 300)
