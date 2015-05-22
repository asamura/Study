import java.util.Scanner;

public class Panel {
	private int row_num;// 세로 줄 수
	private int col_num;// 가로 줄 수
	private int fig_num;
	private Scanner input;
	private int inputValue;
	private boolean open;
	private int[] idx_1;
	private int[] idx_2;
	private int iter;
	private Figure[][] fig;
	private boolean competition;
	private int[][] fig_com;
	private int player_correct;
	private int computer_correct;

	Panel() {
		this(2, 2);
	}

	Panel(int row, int column) {
		this.row_num = row;
		this.col_num = column;
		fig_num = row_num * col_num;
		idx_1 = new int[2];
		idx_2 = new int[2];
		iter = 1;
		competition = false;
		player_correct = 0;
		computer_correct = 0;
	}

	public void initialize_panel() {
		// initialize panel
		fig_num = row_num * col_num;
		fig = new Figure[row_num][col_num];

		// check if the number of figures is an even number
		if (fig_num % 2 != 0) {
			// raise exception
			throw new RuntimeException(
					"The number of figures must be an even number");
		}

		// initialize figure instances in the panel
		for (int i = 0; i < row_num; i++) {
			for (int j = 0; j < col_num; j++) {
				fig[i][j] = new Figure();
			}
		}

		// initialize figure occurrence table
		int[] count = new int[fig_num / 2];
		for (int i = 0; i < count.length; i++) {
			count[i] = 0;
		}

		// put figures to each place in the panel
		for (int i = 0; i < row_num; i++) {
			for (int j = 0; j < col_num; j++) {
				while (true) {
					int temp = (int) (Math.random() * (fig_num / 2));
					// check if good to put
					if (count[temp] < 2) {
						fig[i][j].set_num(temp);
						count[temp]++;
						break;
					}
				}
			}
		}
	}

	private void print_panel() {
		// print column indices
		for (int j = 0; j < col_num; j++) {
			System.out.print("\t" + j); // NOTE '\t' is a tab character
		}
		System.out.println();
		System.out.println();

		// print row indices and figure states
		for (int i = 0; i < row_num; i++) {
			System.out.print(i); // row number
			// state of each figure in the panel
			for (int j = 0; j < col_num; j++) {
				if (fig[i][j].is_open()) {
					System.out.print("\t" + fig[i][j].get_num());

					if (this.competition) {
						// prints who opens this figure
						System.out.print("(" + fig[i][j].get_player() + ")");
					}
				} else {
					System.out.print("\t" + "*");
				}
			}
			System.out.println();
		}
		System.out.println();
		System.out.println();
	}

	public void start_game() {
		while (iter > 0) {
			System.out.println("------------------------------------");
			System.out.println("Try number: " + iter);
			print_panel();

			// User's turn
			System.out.println("Your turn:");
			get_input();
			if (compare()) {
				make_open(false);
			}

			// Check if game is finished
			if (termination()) {
				print_panel();
				System.out.println("------------------------------------");
				break;
			}

			// if you chose competition mode
			if (this.competition) {
				// Computer's turn
				System.out.println("Computer's turn:");
				computer_process();
				if (compare()) {
					make_open(true);
				}
			}

			// Check if game is finished
			if (termination()) {
				print_panel();
				System.out.println("------------------------------------");
				break;
			}

			iter++;
		}
	}

	/*** Methods to be filled ***/
	public void play_with_computer() {
		// TODO: Fill this method
		this.competition = true;
		fig_com = new int[row_num][col_num];
		for (int i = 0; i < fig_com.length; i++) {
			for (int j = 0; j < fig_com[0].length; j++) {
				fig_com[i][j] = -1;
			}
		}
	}

	void get_input() {
		for (int i = 0; i < 2; i++) {
			int j = 0;
			if (i == 0) {
				System.out.println("--- 1st choice ---");
			} else {
				System.out.println("--- 2nd choice ---");
			}
			System.out.print("Choose row index: ");
			Scanner scan1 = new Scanner(System.in);
			input = scan1;
			if (!input_is_correct(j)) {
				System.out
						.println("This card is not correct. Please, try again.");
				get_input();
				return;
			}
			j++;
			int a = inputValue;
			System.out.print("Choose column index: ");
			Scanner scan2 = scan1;
			input = scan2;
			if (!input_is_correct(j)) {
				System.out
						.println("This card is not correct. Please, try again.");
				get_input();
				return;
			}
			int b = inputValue;
			System.out.println("(" + a + ", " + b + ") selected");
			System.out.println("");
			if (fig[a][b].is_open()) {
				System.out
						.println("This card is already opened. Please, try again.");
				get_input();
				return;
			}
			if (i == 0) {
				idx_1[0] = a;
				idx_1[1] = b;
			} else {
				idx_2[0] = a;
				idx_2[1] = b;
			}
		}
		if (idx_1[0] == idx_2[0] && idx_1[1] == idx_2[1]) {
			System.out
					.println("This card is already selected. Please, try again.");
			get_input();
			return;
		}
		if (competition) {
			fig_com[idx_1[0]][idx_1[1]] = fig[idx_1[0]][idx_1[1]].get_num();
			fig_com[idx_2[0]][idx_2[1]] = fig[idx_2[0]][idx_2[1]].get_num();
		}
		System.out.println("fig[" + idx_1[0] + "][" + idx_1[1]
				+ "].get_num(): " + fig[idx_1[0]][idx_1[1]].get_num());
		System.out.println("fig[" + idx_2[0] + "][" + idx_2[1]
				+ "].get_num(): " + fig[idx_2[0]][idx_2[1]].get_num());
		System.out.println("");
	}

	boolean input_is_correct(int row_or_col) {
		if (!input.hasNextInt()) {
			return false;
		}
		inputValue = input.nextInt();
		if (row_or_col == 0) {
			if (inputValue >= row_num) {
				return false;
			}
		} else {
			if (inputValue >= col_num) {
				return false;
			}
		}
		return true;
	}

	private boolean compare() {
		// TODO: Fill this method
		if (fig[idx_1[0]][idx_1[1]].get_num() == fig[idx_2[0]][idx_2[1]]
				.get_num()) {
			return true;
		} else {
			return false;
		}
	}

	private void make_open(boolean is_computer) {
		// TODO: Fill this method
		fig[idx_1[0]][idx_1[1]].set_open();
		fig[idx_2[0]][idx_2[1]].set_open();

		if (is_computer) {
			computer_correct++;
			fig[idx_1[0]][idx_1[1]].set_player("C");
			fig[idx_2[0]][idx_2[1]].set_player("C");
		} else {
			player_correct++;
			fig[idx_1[0]][idx_1[1]].set_player("P");
			fig[idx_2[0]][idx_2[1]].set_player("P");
		}
	}

	private void computer_process() {
		// TODO: Fill this method
		if (compare_test(fig_com)) {
			for (int i = 0; i < row_num; i++) {
				for (int j = 0; j < col_num; j++) {
					for (int l = 0; l < row_num; l++) {
						for (int m = 0; m < col_num; m++) {
							if (fig_com[i][j] == fig_com[l][m]) {
								boolean open1 = fig[i][j].is_open();
								boolean open2 = fig[i][j].is_open();
								if (!open1 && !open2) {
									if (i == l && j == m) {
									} else {
										if (fig_com[i][j] >= 0
												&& fig_com[l][m] >= 0) {
											idx_1[0] = i;
											idx_1[1] = j;
											idx_2[0] = l;
											idx_2[1] = m;
											System.out.println("fig["
													+ idx_1[0]
													+ "]["
													+ idx_1[1]
													+ "].get_num(): "
													+ fig[idx_1[0]][idx_1[1]]
															.get_num());
											System.out.println("fig["
													+ idx_2[0]
													+ "]["
													+ idx_2[1]
													+ "].get_num(): "
													+ fig[idx_2[0]][idx_2[1]]
															.get_num());
											System.out.println("");
											return;
										}
									}
								}
							}
						}
					}
				}
			}
		} else {
			idx_1[0] = (int) (Math.random() * row_num);
			idx_1[1] = (int) (Math.random() * col_num);
			if (fig[idx_1[0]][idx_1[1]].is_open()) {
				computer_process();
				return;
			}
			idx_2[0] = (int) (Math.random() * row_num);
			idx_2[1] = (int) (Math.random() * col_num);
			if (fig[idx_2[0]][idx_2[1]].is_open()) {
				computer_process();
				return;
			}
			if (idx_1[0] == idx_2[0] && idx_1[1] == idx_2[1]) {
				computer_process();
				return;
			}
			fig_com[idx_1[0]][idx_1[1]] = fig[idx_1[0]][idx_1[1]].get_num();
			fig_com[idx_2[0]][idx_2[1]] = fig[idx_2[0]][idx_2[1]].get_num();
			System.out.println("fig[" + idx_1[0] + "][" + idx_1[1]
					+ "].get_num(): " + fig[idx_1[0]][idx_1[1]].get_num());
			System.out.println("fig[" + idx_2[0] + "][" + idx_2[1]
					+ "].get_num(): " + fig[idx_2[0]][idx_2[1]].get_num());
			System.out.println("");
		}
	}

	private boolean compare_test(int[][] result_idx) {
		// TODO: Fill this method
		result_idx = new int[row_num][col_num];
		for (int i = 0; i < row_num; i++) {
			for (int j = 0; j < col_num; j++) {
				for (int l = 0; l < row_num; l++) {
					for (int m = 0; m < col_num; m++) {
						if (i == l && j == m) {
						} else {
							if (fig_com[i][j] == fig_com[l][m]) {
								if (fig_com[i][j] >= 0 && fig_com[l][m] >= 0) {
									return true;
								}
							}
						}
					}
				}
			}
		}
		return false;
	}

	private boolean termination() {
		// TODO: Fill this method
		int a = 0;
		for (int i = 0; i < row_num; i++) {
			for (int j = 0; j < col_num; j++) {
				if (fig[i][j].is_open()) {
					a++;
				}
			}
		}
		if (a == fig_num) {
			if (competition) {
				System.out.println("Player's correct pair: " + player_correct);
				System.out.println("Computer's correct pair: "
						+ computer_correct);
				if (player_correct > computer_correct) {
					System.out.println("Player Win!");
				} else {
					System.out.println("Computer Win!");
				}
				if (player_correct == computer_correct) {
					System.out.println("Draw!");
				}
			} else {
				System.out.println("The number of try: " + iter);
			}
			return true;
		} else {
			return false;
		}
	}
}
