********** UI CLASSES **********
MainActivity
- The main screen of the app. 
- Allows access to LibraryActivity, SearchActivity, 
  InstructionsActivity, and AboutActivity through buttons
- Where test recipes are created through static class RecipeGenerator
  and stored in an ArrayList<Recipe>
- Passes this list of recipes by serializing in intents to LibraryActivity/SearchActivity

LibraryActivity
- The screen where the user views their list of saved recipes
- Can delete recipes from the list by using the Edit button,
  which reveals OK/Cancel buttons and checkboxes beside recipes.
  - User checks recipes they want to delete, and can hit OK to delete them.
  - User can hit Cancel to cancel deletion.
- Can add recipes to the list by pressing the add button which
  creates an empty recipe and opens it in a a RecipeActivity.

RecipeActivity
- The screen where the user views a recipe.
- Can edit a recipe by using the Edit button, which reveals
  OK/Cancel buttons and makes the text fields editable.
  - User can edit desired fields and hit OK to save them.  
  - User can hit Cancel to make fields non-editable again.
- uses a custom class, CustomEditText that extends EditText to
  display recipe information

CustomEditText
- extends EditText
- used to override the onSelectionChanged method so that when a user
  taps on an editable field (such as when editing a recipe), that the cursor
  position be placed at position 0 for String fields, and at position 1 
  for int fields
  - This makes adding a recipe more convenient since the cursor always
    begins where the user would want to start editing

SearchActivity
- The screen where the user searches for recipes. 
- User can enter search terms seperated by boolean operators NOT, AND, OR
- Recipes are returned in order of number of matches with the seach terms.
- e.g., Tomatoes AND Bagel
  - Recipe(s) with both Tomatoes and Bagel will be ranked highest
  - Recipe(s) with Tomatoes only or Bagel only will be ranked next.
    - The above case results in a tie, since recipes with only Tomatoes
      and recipes with only Bagel each have only one matching term, and hence
      they have the same rank.
    - In this case, alphabetical sorting is used, so Bagel recipes rank higher. 
- e.g., Bagel AND NOT Lox
   - Recipe(s) with Bagel and no Lox will rank highest
   - Recipe(s) with either Bagel or recipe(s) with no Lox will be next (with equal rank).

InstructionsActivity
- Displays the instructions for the app by loading them into 
  a webView from a local hmtl file in /assets

AboutActivity
- Displays some info about the app by loading them into a webView 
  from a local html file in /assets.

********** Other CLASSES **********
Recipe
- The model for the app. Contains all attributes for a recipe such as name,
  cooking time, etc.
- Contains a match count attribute for ranking searches
  - initially equal to 0
  - the match count is incremented/decremented through public methods 
    incrementMatchCount/decrementMatchCount when the recipe has a match
    with some search term
  - when the SearchActivity is exited, all match counts are reset
- in RecipeActivity, the fields that display recipe attributes fetch them from
  this model through public getters
- in RecipeActivity, when editing a recipe, this is done by updating the attributes
  in this model through public setters
- extends Serializable to allow for passing recipes to/from Activities through intents
- extends Comparable to allow ranked by match count if match counts are unequal, or 
  by alphabetical order of recipe name if match counts are equal.

SearchEngine
- The class that actually performs recipe searches
- The constructor takes the ArrayList of recipes and the user's search string
- The following steps are then performed, in the constructor, to produce search results:
    1. The user's search string is converted to an ArrayList of individual string tokens
       with the method tokenize()
       e.g., Tomatoes AND NOT Bagel 
       will become the ArrayList {Tomatoes, AND, NOT, Bagel}
    2. The list of tokens is then passed to method insertParenthesesForNOT
       which inserts parentheses tokens around search terms with the NOT operator
       e.g., the above ArrayList becomes {Tomatoes, AND, (, NOT, Bagel, )}
       
       This is done so that we obtain the proper postFix expression in step 4.
    3. We concatenate any tokens that should really be one token with 
       method concatenateMultiWordTokens().

       For example, step 1 will turn the search string "ice cream" into {ice, cream}, 
       when we probably want {ice cream}. The method does this by iterating through the
       list of tokens and comparing the current token with the next one. If the current
       token isn't an operator or parentheses, and the next token isn't an operator or parentheses
       we know it's a multi-word token, and so we concatenate them and remove the individual tokens.
    4. The list of tokens is converted into a postfix expression with method toPostFix().
       e.g., from our example above, {Tomatoes, AND, (, NOT, Bagel, )} becomes
             {Tomatoes, Bagel, NOT, AND}

	     This is done with a stack based algorithm. We iterate through the tokens.
             When a search term is encountered, it is added to an output ArrayList.
             When an operator is encountered, it is pushed to the stack.
             When an open parentheses is encountered, it is pushed to the stack.
             When a closed parentheses is encountered the stack is popped to the output
             ArrayList until a closed parentheses is encountered.
             
             This ensures proper precedence is maintained for the NOT operator.
    5. The list of tokens in postfix form is evaluated through another stack based algorithm
       with method evaluate(), that returns a HashSet<Recipe> of search results.
       It pushes tokens to the stack until an operator is encountered, at which point it
       pops the operands (the search tokens) into an ArrayList and sends them to another method
       along with the operator token to method performOperation() which returns the result
       and adds it to the hash set.
    6. Individual operations are performed in method performOperation().

       For AND/OR operations, we check every recipe against every token in the operands list
       for a match. Whenever there is a match, the recipe's match count is incremented, and it is
       added to a HashSet<Recipe>. This ensures no duplicate entries. 

       For NOT operations we check every recipe against every token again, but look for Recipes
       that do not have a match instead. Similarly, we increment match count and add the recipe
       to the HashSet if that is the case.
       Alternatively, if a recipe does have a match with the token, the match count is decremented
       to lower its rank in the search results.

RecipeGenerator
- This is a static class with one method, generateRecipe() that creates a bunch of
  Recipes and adds them to an ArrayList<Recipe>, which it returns. This is used to load
  a list of example recipes to demonstrate the app's functionality.


   