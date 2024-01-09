# about-PatchMapping

@PatchMapping annotation in SpringBoot

Partial update
The PUT method updates the whole record. There may be a scenario when only one or two fields needs to be updated. In that case, sending the whole record does not make sense. The HTTP PATCH method is used for partial updates.

Sometimes we may need to update a single field. For example, once we enter a player in our database, the field that will most likely change is his titles count. The player entity only has a few fields and PUT can be used for update. But if the entity is large and contains nested objects, it will have a performance impact to send the whole entity only to update a single field.
_____________________________________________________________________________________________________________________________________________________________________________________________________________________
In the PlayerService class, we will implement a method to handle partial updates to the Player object. The method patch will have two arguments. The first is the id of the player on which the patch is to be applied. The second argument is the Map containing the key-value pairs of the fields that will be updated. The key (field name) is a String while the value is an Object as it can have different datatypes.

public Player patch( int id, Map<String, Object> playerPatch) {

}
Inside the method, we will use the id to fetch the existing Player object from the database using the findById method of the JpaRepository. This method loads the entity from the database unlike the getOne method, which returns a proxy object without hitting the database. The findById returns an Optional and we need to check if a Player object is returned using the isPresent() method.

public Player patch( int id, Map<String, Object> playerPatch) {

    Optional<Player> player = repo.findById(id);
    if (player.isPresent()){
        //update fields using Map
    }
    return repo.save(player);               
}
Using reflection
Next, we will loop through the Map, find the field that will be updated, and then change the value of that field in the existing Player object that we retrieved from the database in the previous step. The Reflection API is used to examine and modify fields, methods, and classes at runtime. It allows access to the private fields of a class and can be used to access the fields irrespective of their access modifiers. Spring provides the ReflectionUtils class for handling reflection and working with the Reflection API.

The ReflectionUtils class has a findField method to identify the field of an object using a String name. The findField method takes two arguments, the class having the field and the name of the field which in our case is contained in the variable key. This method will return a Field object.

Field field = ReflectionUtils.findField(Player.class, key);
To set a value for this field, we need to set the field’s accessible flag to true. ReflectionUtils setAccessible method, when called on a field, toggles its accessible flag. We can also use another method called makeAccessible. This method makes the given field accessible by calling the setAccessible(true) method if necessary.

ReflectionUtils.makeAccessible(field);
Lastly, we will call the setField method and use the value from the Map to set the field in the player object. The setField method takes three arguments, the reference of the field, the object in which the field is to be set, and the value to set. This method requires that the given field is accessible.

ReflectionUtils.setField(field, player.get(), value);
Here, we have used the get method on the Optional player object to retrieve it.

In this way using reflection, a field can be updated in an object. Since we are passing the fields to be updated as a Map, we will use the above steps while iterating through the map of key-value pairs as follows:

playerPatch.forEach( (key, value) -> {
    Field field = ReflectionUtils.findField(Player.class, key);
    ReflectionUtils.makeAccessible(field);
    ReflectionUtils.setField(field, player.get(), value);
});
This code will iterate through the Map and make desired changes in the player object. At the end we will call the save method to update the player record. The complete code of the method is shown below:

public Player patch( int id, Map<String, Object> playerPatch) {

    Optional<Player> player = repo.findById(id);
    
    if(player.isPresent()) {            
        playerPatch.forEach( (key, value) -> {
            Field field = ReflectionUtils.findField(Player.class, key);
            ReflectionUtils.makeAccessible(field);
            ReflectionUtils.setField(field, player.get(), value);
        });
    }
    return repo.save(player.get());             
}
