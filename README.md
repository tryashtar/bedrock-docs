### Bedrock Docs
[The official documentation for Bedrock Minecraft](https://docs.microsoft.com/en-us/minecraft/creator/), like Bedrock itself, is terrible, incomplete, frustrating, and unusable.

Every page I read leaves me with a dozen unanswered questions -- extremely obvious questions that any documentation writer with a brain would have known to answer.

Ultimately, I begrudgingly boot up the game and test to find out for myself. The results of my experiments have until now have existed only in my mind (and in the minds of every competent Bedrock mapmaker). Let's put them here instead.

This will be very slow. My process is something like this:
* Read the official docs on the subject
* Look how vanilla does it
* Check the game disassembly when necessary
* Assume everything I read is a lie, verify it in-game
* Think "I wonder what would happen if..." and run tests until I fully understand the capabilities, behavior, and bugs. Tests like:
  * What if I make this value negative?
  * Does this take an array or just a single value?
  * Does this accept molang?
  * Does this work differently on the player?
  * What happens if you relog?
  * Do damage values work here?
  * What happens when you remove this component?
  * When exactly does this give a content log error?
