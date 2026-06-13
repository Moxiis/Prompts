	- Role - who are you

	- Prompt do modelu nie powinien być za długi i powienien być łatwy do zrozumienia
	
	- Kod ma być:
		○ Clean
		○ minimalistyczny
		○ Lean
		○ Efficient
		○ Well structured
		○ Enforce software best practices (KISS, SOLID, DRY, YAGNI) and MLOps standards (reproducibility, modularity, and environment isolation).
		
	- DOD(Definition of done)
	- Jeśli testy, jakiego środowiska używać
	- Zooptymalizwowany output prompt: prompty powinny być treściwe i zooptymalizowane pod względem treści i użycia tokenów, nie zalewać użytkownika nic nie znaczącymi terminami, rzeczy w nich wypisane nie powinny się powtarzać. Powinien być napisany w prosty i czytelny sposób
	- Additionaly if you see that my suggestion is wrong and could make our codebase wrong, don't implement this change right away and first try to explain to me why my way of thought is wrong and what would be a best aproach
	- Don't hardcode anything
	- Szukaj #AGENT w kodzie, oznacza to że to zadanie/komentarz jest przeznaczone dla agenta:
	
	# AGENT:
	# Problem: ...
	# Change: ...
	# Constraints: ...
	# Example (optional): ...
	
	
	


	- Planning Framework (The "Blueprint"): Your output must be saved to .ai/initial_plan.md and follow this structure[Optional]

Update 26.05:

	- Dodać informacje o możliwości zadawania pytań:
		○ If you have some questions related to my changes feel free to ask. it is better to make clear some aspects than to fix them in later iterations
	- Dodać plik z głównymi wytycznymi dla agenta CLAUD/COPILOT.MD:
		○ Dodać możliwość ich modyfikcji poprzez ciągłą pracę z agentem, aby ciągle go ulepszać na postawie preferencji uzytkownika


Update 09.06:
	- Dodać info aby marnował jak najmniej tokenów na odpowiedż i aby był tooken efficient
