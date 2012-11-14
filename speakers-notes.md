	@Controller
	@RequestMapping("/games")

---

	@RequestMapping(method = RequestMethod.POST, value = "")
	ResponseEntity<Void> createGame() {
	    Game game = this.gameRepository.create();

	    HttpHeaders headers = new HttpHeaders();
	    headers.setLocation(linkTo(GamesController.class).slash(game.getId()).toUri());

	    return new ResponseEntity<Void>(headers, HttpStatus.CREATED);
	}


* `Location` header
* `CREATED` return code

---

	@RequestMapping(method = RequestMethod.GET, value = "/{gameId}", produces = { MediaType.APPLICATION_JSON_VALUE, MediaType.TEXT_XML_VALUE })
	ResponseEntity<GameResource> showGame(@PathVariable Long gameId) throws GameDoesNotExistException {
	    Game game = this.gameRepository.retrieve(gameId);
	    GameResource resource = this.gameResourceAssembler.toResource(game);

	    return new ResponseEntity<GameResource>(resource, HttpStatus.OK);
	}

* `produces`
* **SINGLE `produces`**
* **DON'T use resources**

---

	@RequestMapping(method = RequestMethod.PUT, value = "/{gameId}/doors/{doorId}", consumes = MediaType.APPLICATION_JSON_VALUE, produces = {
	    MediaType.APPLICATION_JSON_VALUE, MediaType.TEXT_XML_VALUE })
	ResponseEntity<Void> modifyDoor(@PathVariable Long gameId, @PathVariable Long doorId, @RequestBody Map<String, String> body)
	    throws MissingKeyException, GameDoesNotExistException, IllegalTransitionException, DoorDoesNotExistException {
	    DoorStatus status = getStatus(body);
	    Game game = this.gameRepository.retrieve(gameId);

	    if (DoorStatus.SELECTED == status) {
	        game.select(doorId);
	    } else if (DoorStatus.OPEN == status) {
	        game.open(doorId);
	    } else {
	        throw new IllegalTransitionException(gameId, doorId, status);
	    }

	    return new ResponseEntity<Void>(HttpStatus.OK);
	}

* `consumes`

---

	@ExceptionHandler({ GameDoesNotExistException.class, DoorDoesNotExistException.class })
	ResponseEntity<String> handleNotFounds(Exception e) {
	    return new ResponseEntity<String>(e.getMessage(), HttpStatus.NOT_FOUND);
	}

	@ExceptionHandler({ IllegalArgumentException.class, MissingKeyException.class })
	ResponseEntity<String> handleBadRequests(Exception e) {
	    return new ResponseEntity<String>(e.getMessage(), HttpStatus.BAD_REQUEST);
	}

	@ExceptionHandler(IllegalTransitionException.class)
	ResponseEntity<String> handleConflicts(Exception e) {
	    return new ResponseEntity<String>(e.getMessage(), HttpStatus.CONFLICT);
	}

* `@ExceptionHandler` can handle single or multiple types
* Useful information in the response body

---

	GameResource resource = createResource(game);
	resource.status = game.getStatus();
	resource.add(linkTo(GamesController.class).slash(game).slash("doors").withRel("doors"));
	return resource;

* `ResourceSupport` (don't want to pollute domain model)
* `ResourceAssemblerSupport`
* `linkTo` based on Controller `@RequestMapping`

---

	DoorResource resource = new DoorResource();
	resource.status = door.getStatus();
	resource.content = door.getContent();
	resource.add(linkTo(GamesController.class).slash(game).slash("doors").slash(door).withSelfRel());

	return resource;

* Extension of `ResourceAssemblerSupport` not required
* Have to manually create instance and add self rel
* **USE Resource where avoided earlier**
