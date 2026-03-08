# ChatGPT API Integration Documentation

## Overview
PantryAI uses the official OpenAI ChatGPT API (gpt-4o-mini model) to generate real, creative recipes from user-provided leftover ingredients.

## API Configuration

### Environment Variables
The OpenAI API key is securely stored in Supabase Edge Function secrets:
- **Variable Name**: `OPENAI_API_KEY`
- **Current Key**: Configured and deployed

### Model Used
- **Model**: `gpt-4o-mini`
- **Temperature**: 0.7 (balanced creativity)
- **Max Tokens**: 1000

## How It Works

### 1. User Input
Users enter their leftover ingredients through a tag-based input interface on the Recipe Generator page.

### 2. API Request
When the user clicks "Generate Recipe with AI":
1. Frontend calls the `generateRecipe()` function in `@/db/api.ts`
2. The function invokes the Supabase Edge Function `generate-recipe`
3. Edge Function sends a structured prompt to ChatGPT API

### 3. ChatGPT Prompt Structure
```
You are a professional chef. A user has the following leftover ingredients: [INGREDIENT LIST].

Create a realistic recipe using these ingredients.

Return the result in this structured JSON format (return ONLY valid JSON, no additional text):
{
  "title": "Recipe name",
  "ingredients": ["ingredient 1 with quantity", "ingredient 2 with quantity", ...],
  "instructions": "Step-by-step cooking instructions as a single string with numbered steps",
  "cookingTime": "Estimated time (e.g., '30 minutes')",
  "costSavings": 5.50
}
```

### 4. Response Processing
- Edge Function receives ChatGPT's response
- Extracts JSON from the response
- Validates required fields (title, ingredients, instructions)
- Saves recipe to Supabase database
- Updates user's impact statistics
- Returns recipe to frontend

### 5. Error Handling
**IMPORTANT**: The system NEVER generates fake or placeholder recipes.

If the API fails:
- Error is logged with details
- User sees error message: "Failed to generate recipe from ChatGPT API"
- No recipe is displayed
- User can try again

## API Endpoint
```
POST https://api.openai.com/v1/chat/completions

Headers:
- Authorization: Bearer {OPENAI_API_KEY}
- Content-Type: application/json

Body:
{
  "model": "gpt-4o-mini",
  "messages": [
    {
      "role": "system",
      "content": "You are a professional chef..."
    },
    {
      "role": "user",
      "content": "[Prompt with ingredients]"
    }
  ],
  "temperature": 0.7,
  "max_tokens": 1000
}
```

## Response Format
ChatGPT returns a JSON object with:
- `title`: Recipe name
- `ingredients`: Array of ingredients with quantities
- `instructions`: Step-by-step cooking instructions
- `cookingTime`: Estimated cooking time
- `costSavings`: Estimated cost savings (number)

## Database Storage
Generated recipes are stored in the `recipes` table:
```sql
CREATE TABLE recipes (
  id uuid PRIMARY KEY,
  user_id uuid REFERENCES profiles(id),
  title text NOT NULL,
  ingredients jsonb NOT NULL,
  instructions text NOT NULL,
  cooking_time text,
  cost_savings numeric(10,2),
  created_at timestamptz DEFAULT now()
);
```

## User Experience

### Visual Indicators
- Landing page badge: "Powered by ChatGPT"
- Recipe Generator page: "Generate Recipe with ChatGPT"
- Loading state: "Generating Recipe with ChatGPT..."
- Generated recipes show a "ChatGPT" badge

### Success Flow
1. User adds ingredients
2. Clicks "Generate Recipe with AI"
3. Loading animation appears
4. ChatGPT generates recipe (typically 2-5 seconds)
5. Recipe appears with ChatGPT badge
6. Success toast: "Recipe generated successfully from ChatGPT!"

### Error Flow
1. User adds ingredients
2. Clicks "Generate Recipe with AI"
3. Loading animation appears
4. API fails (network, quota, invalid key, etc.)
5. Error toast: "Failed to generate recipe from ChatGPT API..."
6. Console logs detailed error information
7. No fake recipe is shown

## Security
- API key stored securely in Supabase Edge Function environment
- Never exposed to client-side code
- All API calls go through Edge Function (server-side)
- CORS headers properly configured

## Testing
To test the integration:
1. Sign up/login to PantryAI
2. Navigate to "Generate Recipe"
3. Add ingredients (e.g., "chicken", "rice", "broccoli")
4. Click "Generate Recipe with AI"
5. Verify real recipe appears from ChatGPT
6. Check console for any errors

## Troubleshooting

### Common Issues
1. **"OpenAI API key not configured"**
   - Check that OPENAI_API_KEY is set in Supabase secrets
   - Redeploy Edge Function

2. **"Failed to generate recipe from ChatGPT API"**
   - Check API key validity
   - Check OpenAI account quota/billing
   - Check network connectivity
   - Review Edge Function logs

3. **"Failed to parse recipe data"**
   - ChatGPT returned invalid JSON
   - Prompt may need adjustment
   - Try again (ChatGPT responses can vary)

## Files Modified
- `/supabase/functions/generate-recipe/index.ts` - Edge Function with OpenAI integration
- `/src/db/api.ts` - Enhanced error handling
- `/src/pages/RecipeGenerator.tsx` - Updated UI with ChatGPT branding
- `/src/pages/Landing.tsx` - Updated to mention ChatGPT

## API Costs
- Model: gpt-4o-mini
- Cost: ~$0.15 per 1M input tokens, ~$0.60 per 1M output tokens
- Average recipe generation: ~500 tokens total
- Estimated cost per recipe: ~$0.0003 (very affordable)

## Compliance
- Uses official OpenAI API
- No fake or generated data
- Real-time API calls only
- Proper error handling
- User-friendly error messages
