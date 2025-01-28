<script lang="ts">
	let formula: string = $state('');

	const addToFormula = (input: string) => {
		// test for number beginning with multiple 0 but allow for decimal values
		if (!/^0+/.test(formula) && !(input === '.' && formula[formula.length - 1] === '.')) {
			// check if the last number is a floating point number
			// and if it is and the user is trying to enter another '.' don't register it
			// prevents: e.g. 5.5.5
			if (!(/-?\d*\.\d+$/.test(formula) && input === '.')) {
				// handle consecutive operators
				if (/[+*\/]$/.test(input) && /[+*-\/]$/.test(formula[formula.length - 1])) {
					// if the element before the last input was an operator (apart from -)
					// and the current input is also an operator
					// then replace the previous one with the current one
					// solves: e.g. 5 * - + 5
					if (/[+*\/]/.test(formula[formula.length - 2])) {
						formula = formula.replace(/..$/, input);
					} else {
						// if the current input is an operator
						// and the previous input was also an operator
						// then just replace the previous operator
						formula = formula.replace(/.$/, input);
					}
				} else {
					formula += input;
				}
			}
		}
	};

	let result: number = 0;

	const computeResult = () => {
		// check if we need to operate on the result from the previous value
		// if yes, then modify the formula to include the result
		if (/[*\/+-]/.test(formula[formula.length - 1])) {
			formula = result.toString() + formula;
		}

		result = 0;

		// get all the numbers (including negative numbers)
		let numbers: number[] | undefined = formula.match(/-?\d*\.?\d+|\d+/g)?.map((e) => Number(e));
		const operators: string[] | undefined = formula
			.match(/[+\-*\/]/g)
			// filter out operators that are from negative numbers (e.g. 2*-5 -> [*] and not [*, -])
			?.filter((e, i, arr) => !(e === '-' && (arr[i - 1] === '*' || arr[i - 1] === '/')));

		if (numbers !== undefined && operators !== undefined) {
			// perform multiplication and division first (precedence)
			// take the numbers around the operator
			// and save the result (product) in the left number
			// then delete the right number
			for (let i = 0; i < operators.length; i++) {
				if (operators[i] === '*') {
					numbers[i] = numbers[i] * numbers[i + 1]; // use values around operator
					numbers.splice(i + 1, 1); // remove the number after use
					operators.slice(i, 1); // get rid of the operator after use
				} else if (operators[i] === '/') {
					numbers[i] = numbers[i] / numbers[i + 1];
					numbers.splice(i + 1, 1);
					operators.slice(i, 1);
				}
			}

			// all the operators that are left are "+" and "-"
			for (let i = 0; i < operators.length; i++) {
				if (operators[i] === '+') {
					numbers[i] = numbers[i] + numbers[i + 1];
					numbers.splice(i + 1, 1);
					operators.splice(i, 1);
				} else if (operators[i] === '-') {
					// if the number is positive
					// process the result as usual
					// solves: e.g. 2 - -5
					if (numbers[i + 1] >= 0) {
						numbers[i] = numbers[i] - numbers[i + 1];
					} else {
						// if the number is negative, account for that when processing
						numbers[i] = numbers[i] - -numbers[i + 1];
					}
					numbers.splice(i + 1, 1);
					operators.splice(i, 1);
				}
			}
			// add all numbers that may be left together to get the result
			numbers.map((e) => (result += e));
			formula = result.toString(); // update formula to reflect result
		}
	};

	const clearAll = () => {
		formula = '';
	};
</script>

<div>
	<div class="calc-container">
		<div class="layout-container">
			<div class="display-container">
				<div id="display">{formula !== '' ? formula : '0'}</div>
			</div>

			<div class="btn-container">
				<div class="numbers">
					<button onclick={clearAll} id="clear">AC</button>
					<button onclick={() => addToFormula('7')} id="seven">7</button>
					<button onclick={() => addToFormula('8')} id="eight">8</button>
					<button onclick={() => addToFormula('9')} id="nine">9</button>
					<button onclick={() => addToFormula('4')} id="four">4</button>
					<button onclick={() => addToFormula('5')} id="five">5</button>
					<button onclick={() => addToFormula('6')} id="six">6</button>
					<button onclick={() => addToFormula('1')} id="one">1</button>
					<button onclick={() => addToFormula('2')} id="two">2</button>
					<button onclick={() => addToFormula('3')} id="three">3</button>
					<button onclick={() => addToFormula('0')} id="zero">0</button>
					<button onclick={() => addToFormula('.')} id="decimal">.</button>
				</div>
				<div class="operators">
					<button onclick={() => addToFormula('+')} id="add">+</button>
					<button onclick={() => addToFormula('-')} id="subtract">-</button>
					<button onclick={() => addToFormula('*')} id="multiply">*</button>
					<button onclick={() => addToFormula('/')} id="divide">/</button>
					<button onclick={computeResult} id="equals">=</button>
				</div>
			</div>
		</div>
	</div>
	<p class="description">
		Made by <a href="https://github.com/turtureanu">@turtureanu</a> in Svelte as a freeCodeCamp certification
		project. The code is weird, but the project requirments are also really weird.
	</p>
</div>

<style lang="scss">
	.layout-container {
		width: 100%;
	}
	button {
		cursor: pointer;
		border-radius: 0;
		border: 0;
		width: 5rem;
		font-size: 1.25rem;
		font-weight: 500;
		height: 5rem;
		outline: solid 3px hsl(0, 0%, 97%);
		outline-offset: -3px;
		padding: 0;
		display: block;

		&:hover {
			background-color: hsl(0, 0%, 85%);
		}
	}

	.display-container {
		background-color: #333;
	}

	.description {
		width: (4 * 5)-1rem;
		background-color: #f0f0f0;
		padding: 1rem;
		line-height: 2.5ch;
		font-size: 0.875rem;
	}

	.calc-container {
		margin: 0 auto;
		background-color: hsl(0, 0%, 81%);
		padding: 0.5em;
		display: flex;
	}

	#display {
		&:last-child {
			justify-self: end;
		}
		overflow-x: auto;
		scrollbar-width: thin;
		padding: 1em 0;
		margin: 0 auto;
		width: (4 * 5)-2rem;
		text-align: right;
		font-size: 1.8rem;
		max-width: 100%;
		color: white;
	}

	.numbers {
		display: grid;
		grid-template-columns: repeat(3, 1fr);

		#zero {
			width: 100%;
			grid-column: 1 / 3;
		}

		#decimal {
			grid-column: 3;
		}
	}

	.btn-container {
		display: flex;
	}

	#clear {
		grid-column: 1 / -1;
		background-color: hsl(0, 51%, 80%);
		width: 100%;

		&:hover {
			background-color: hsl(0, 51%, 75%);
		}
	}

	.operators {
		display: flex;
		flex-direction: column;
	}
</style>
