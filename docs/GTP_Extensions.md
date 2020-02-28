## KataGo GTP Extensions

In addition to a basic set of [GTP commands](https://www.lysator.liu.se/~gunnar/gtp/), KataGo supports a few additional commands:

   * `rectangular_boardsize X Y`
      * Sets the board size to a potentially non-square size, width `X` and height `Y`. KataGo's official neural nets are currently not actually trained with non-square sizes, but they actually seem to generalize to them pretty well.
   * `clear_cache`
      * Clears the search tree and the NN cache. Can be used to force KataGo to re-search a position freshly, re-randomizing the search on that position, or to free up memory.
   * `stop`
      * Halts any ongoing pondering, if pondering was enabled in the gtp config.
   * `kata-get-rules`
      * Returns a JSON dictionary indicating the current rules KataGo is using.
      * For example: `{"hasButton":false,"ko":"POSITIONAL","scoring":"AREA","suicide":true,"tax":"NONE","whiteHandicapBonus":"N-1"}`
      * See https://lightvector.github.io/KataGo/rules.html for a detailed description of the rules implemented.
      * Explanation of individual fields:
         * `ko: ("SIMPLE" | "POSITIONAL" | "SITUATIONAL")` - The rule used for preventing cycles.
         * `scoring: ("AREA" | "TERRITORY")` - The rule used for computing the score of the game.
         * `tax: ("NONE" | "SEKI" | "ALL")` - Modification to the scoring rule, indicating whether territory in SEKI is taxed, or whether ALL groups pay a tax of up to 2 points for eyes.
         * `suicide: (true | false)` - Whether multi-stone suicide is legal.
         * `hasButton: (true | false)` - Whether [button Go](https://senseis.xmp.net/?ButtonGo) is being used.
         * `whiteHandicapBonus` ("0" | "N-1" | "N") - In handicap games, whether white gets 0, N-1, or N bonus points, where N is the number of black handicap stones.
      * **NOTE: It is possible that more fields and more options for these fields will be added in the future.**
   * `kata-set-rules RULES`
      * Sets the current rules KataGo should be using. Does NOT otherwise affect the board position.
      * `RULES` should either be a JSON dictionary in the same format of `kata-get-rules`, or be a shorthand string like `tromp-taylor`. Some possible shorthand strings are:
         * `tromp-taylor  : Equivalent to {"hasButton":false,"ko":"POSITIONAL", "scoring":"AREA",     "suicide":true, "tax":"NONE","whiteHandicapBonus":"0"}`
         * `chinese       : Equivalent to {"hasButton":false,"ko":"SIMPLE",     "scoring":"AREA",     "suicide":false,"tax":"NONE","whiteHandicapBonus":"N"}`
         * `chinese-ogs   : Equivalent to {"hasButton":false,"ko":"POSITIONAL", "scoring":"AREA",     "suicide":false,"tax":"NONE","whiteHandicapBonus":"N"}`
         * `chinese-kgs   : Equivalent to {"hasButton":false,"ko":"POSITIONAL", "scoring":"AREA",     "suicide":false,"tax":"NONE","whiteHandicapBonus":"N"}`
         * `japanese      : Equivalent to {"hasButton":false,"ko":"SIMPLE",     "scoring":"TERRITORY","suicide":false,"tax":"SEKI","whiteHandicapBonus":"0"}`
         * `korean        : Equivalent to {"hasButton":false,"ko":"SIMPLE",     "scoring":"TERRITORY","suicide":false,"tax":"SEKI","whiteHandicapBonus":"0"}`
         * `stone-scoring : Equivalent to {"hasButton":false,"ko":"SIMPLE",     "scoring":"AREA",     "suicide":false,"tax":"ALL", "whiteHandicapBonus":"0"}`
         * `aga           : Equivalent to {"hasButton":false,"ko":"SITUATIONAL","scoring":"AREA",     "suicide":false,"tax":"NONE","whiteHandicapBonus":"N-1"}`
         * `bga           : Equivalent to {"hasButton":false,"ko":"SITUATIONAL","scoring":"AREA",     "suicide":false,"tax":"NONE","whiteHandicapBonus":"N-1"}`
         * `new-zealand   : Equivalent to {"hasButton":false,"ko":"SITUATIONAL","scoring":"AREA",     "suicide":true, "tax":"NONE","whiteHandicapBonus":"0"}`
         * `aga-button    : Equivalent to {"hasButton":true, "ko":"SITUATIONAL","scoring":"AREA",     "suicide":false,"tax":"NONE","whiteHandicapBonus":"N-1"}`
      * KataGo does NOT claim that the above rules are _exactly_ a match. These are merely the _closest_ settings that KataGo has to those countries' rulesets.
      * A small number of combinations are currently not supported by even the latest neural nets, for example `scoring TERRITORY` and `hasButton true`.
      * Older neural nets for KataGo will also not support many of the options, and setting these rules will fail if these neural nets are being used.
      * Note the distinction between the `chinese` and `chinese-ogs, chinese-kgs`. Often in Chinese tournaments, contrary to their nominal written rules, which specify positional superko, only simple ko is used, and triple ko typically does result in the referree voiding the game. However, many servers have implemented the nominal written rules rather than the actually-used rules.
   * `kata-set-rule RULE VALUE`
      * Sets a single field of the current rules, leaving other fields unaffected.
      * For example, `kata-set-rule ko SIMPLE`.
      * May fail, if setting this field would result in a combination of rules that is not supported by the current neural net.
   * `kgs-rules RULES`
      * This is an extension for playing on KGS, via kgsGtp.
      * As specified by kgsGtp docs, `RULES` should be one of `chinese | aga | japanese | new_zealand`.
      * For this extension, `chinese` actually maps to `chinese-kgs` above. Otherwise, has the same effect as `kata-set-rules`.
   * `kgs-time-settings KIND ...`
      * This is an extension for playing on KGS, via kgsGtp.
      * As specified by kgsGtp docs, `KIND` should be one of `none | absolute | canadian | byoyomi`.
         * `none` indicates no time control
         * `absolute` should be followed by a single float `MAINTIME` specifying the time in seconds.
         * `canadian` should be followed by `MAINTIME BYOYOMITIME BYOYOMISTONES` (float,float,int) specifying the main time, the length of the canadian overtime period, and the number of stones that must be played in that period.
         * `byoyomi` should be followed by `MAINTIME BYOYOMITIME BYOYOMIPERIODS` (float,float,int) specifying the main time, the length of each byo-yomi period, and the number of byo-yomi periods.
      * NOTE: As a hack, KGS also expects that when using `byoyomi`, the controller should provide the number of periods left in place of the number of stones left for the GTP `time-settings` command.
   * `lz-analyze [player (optional)] [interval (optional)] KEYVALUEPAIR KEYVALUEPAIR ...`
      * Begin searching and optionally outputting live analysis to stdout. Assumes the normal player to move next unless otherwise specified.
      * Possible key-value pairs:
         * `interval CENTISECONDS` - Output a line every this many centiseconds. Alternate way to specify interval besides as the second argument.
         * `minmoves N` - Output stats for at least N different legal moves if possible (will likely cause KataGo to output stats on 0-visit moves)
         * `maxmoves N` - Output stats for at most N different legal moves (NOTE: Leela Zero does NOT currently support this field)
         * `allow ...` - Not currently implemented in KataGo.
         * `avoid ...` - Not currently implemented in KataGo.
      * Output format:
         * Outputted lines look like `info move E4 visits 1178 winrate 4802 prior 2211 lcb 4781 order 0 pv E4 E3 F3 D3 F4 P4 P3 O3 Q3 O4 K3 Q6 S6 E16 E17 info move P16 visits 1056 winrate 4796 prior 2206 lcb 4769 order 1 pv P16 P17 O17 Q17 O16 E16 E17 F17 D17 F16 K17 D14 B14 P3 info move E3 visits 264 winrate 4752 prior 944 lcb 4722 order 2 pv E3 D5 P16 P17 O17 Q17 O16 E17 H17 D15 C15 D14 info move E16 visits 262 winrate 4741 prior 1047 lcb 4709 order 3 pv E16 P4 P3 O3 Q3 O4 P16 P17 O17 Q17 O16 Q14`
         * `info` - Indicates the start of information for a new possible move
         * `move` - The move being analyzed.
         * `visits` - The number of visits invested into the move so far.
         * `winrate` - 10000 times the winrate of the move so far, rounded to an integer in [0,10000].
         * `prior` - 10000 times the policy prior of the move, rounded to an integer in [0,10000].
         * `lcb` - 10000 times the [LCB](https://github.com/leela-zero/leela-zero/issues/2282) of the move, rounded to an integer in [0,10000].
         * `order` - KataGo's ranking of the move. 0 is the best, 1 is the next best, and so on.
         * `pv` - The principal variation following this move. May be of variable length or even empty.
      * All output values are from the perspective of the current player, unless otherwise configured in KataGo's gtp config.
      * This command will terminate upon any new GTP command being received, as well as upon a raw newline being received, including outputting the usual double-newline that signals a completed GTP response.

   * `kata-analyze [player (optional)] [interval (optional)] KEYVALUEPAIR KEYVALUEPAIR ...`
      * Same as `lz-analyze` except a slightly different output format and some additional options and fields.
      * Additional possible key-value pairs:
         * `ownership true` - Output the predicted final ownership of every point on the board.
      * Output format:
         * **NOTE: Consumers of this data should attempt to be robust to the order of these fields, as well as to possible addition of new fields in the future.**
         * Outputted lines look like `info move E4 visits 487 utility -0.0408357 winrate 0.480018 scoreMean -0.611848 scoreStdev 24.7058 scoreLead -0.611848 scoreSelfplay -0.515178 prior 0.221121 lcb 0.477221 utilityLcb -0.0486664 order 0 pv E4 E3 F3 D3 F4 P4 P3 O3 Q3 O4 K3 Q6 S6 E16 E17 info move P16 visits 470 utility -0.0414945 winrate 0.479712 scoreMean -0.63075 scoreStdev 24.7179 scoreLead -0.63075 scoreSelfplay -0.5221 prior 0.220566 lcb 0.47657 utilityLcb -0.0502929 order 1 pv P16 P17 O17 Q17 O16 E17 H17 D15 C15 D14 C13 D13 C12 D12 info move E16 visits 143 utility -0.0534071 winrate 0.474509 scoreMean -0.729858 scoreStdev 24.7991 scoreLead -0.729858 scoreSelfplay -0.735747 prior 0.104652 lcb 0.470674 utilityLcb -0.0641425 order 2 pv E16 P4 P3 O3 Q3 O4 E3 H3 D5 C5`
         * `info` - Indicates the start of information for a new possible move
         * `move` - The move being analyzed.
         * `visits` - The number of visits invested into the move so far.
         * `winrate` - The winrate of the move so far, as a float in [0,1].
         * `scoreMean` - Same as scoreLead. "Mean" is a slight misnomer, but this field exists to preserve compatibility with existing tools.
         * `scoreStdev` - The predicted standard deviation of the final score of the game after this move, in points. (NOTE: due to the mechanics of MCTS, this value will be **significantly biased high** currently, although it can still be informative as *relative* indicator).
         * `scoreLead` - The predicted average number of points that the current side is leading by (with this many points fewer, it would be an even game).
         * `scoreSelfplay` - The predicted average value of the final score of the game after this move from low-playout noisy selfplay, in points. (NOTE: users should usually prefer scoreLead, since scoreSelfplay may be biased by the fact that KataGo isn't perfectly score-maximizing).
         * `prior` - The policy prior of the move, as a float in [0,1].
         * `utility` - The utility of the move, combining both winrate and score, as a float in [-C,C] where C is the maximum possible utility.
         * `lcb` - The [LCB](https://github.com/leela-zero/leela-zero/issues/2282) of the move's winrate, as a float in [0,1].
         * `utilityLcb` - The LCB of the move's utility.
         * `order` - KataGo's ranking of the move. 0 is the best, 1 is the next best, and so on.
         * `pv` - The principal variation following this move. May be of variable length or even empty.
         * `ownership` - If `ownership true` was provided, then BoardHeight*BoardWidth many consecutive floats in [-1,1] separated by spaces, predicting the final ownership of every board location from the perspective of the current player. Floats are in row-major order, starting at the top-left of the board (e.g. A19) and going to the bottom right (e.g. T1).
  * `lz-genmove_analyze [player (optional)] [interval (optional)] KEYVALUEPAIR KEYVALUEPAIR`
     * Same as `genmove` except will also produce `lz-analyze`-like output while the move is being searched.
     * Behaves like a normal synchronous GTP command, unlike `lz-analyze`. Will NOT terminate prematurely due a newline or addiional GTP command.
     * The final move made will be reported as a single line `play <vertex or "pass" or "resign">`, followed by the usual double-newline.
  * `kata-genmove_analyze [player (optional)] [interval (optional)] KEYVALUEPAIR KEYVALUEPAIR`
     * Same as `lz-genmove_analyze` except with the options and fields of `kata-analyze` rather than `lz-analyze`
  * `kata-raw-nn SYMMETRY`
     * `SYMMETRY` should be an integer from 0-7 or "all".
     * Reports the result of a raw neural net evaluation from KataGo, or multiple raw evaluations in the case of "all".
     * Output format is of the form `SYMMETRY <integer 0-7> <key> <value(s)> <key> <value(s)> ...`, possibly with additional whitespace or newlines between any tokens. In the case of "all", multiple such outputs of this form are concatenated together.
     * Possible keys are currently
     ```
     whiteWin (1 float)  - believed probability that white wins
     whiteLoss (1 float) - believed probability that black wins
     noResult (1 float)  - believed probability of noresult due to triple ko
     whiteLead (1 float) - predicted number of points that white is ahead by (this is the preferred score value for user display).
     whiteScoreSelfplay (1 float) - predicted mean score that would result from low-playout noisy selfplay (may be biased, Kata isn't fully score-maximizing).
     whiteScoreSelfplaySq (1 float) - predicted mean square of score that would result via low-playout noisy selfplay
     policy (boardXSize * boardYSize floats, including possibly NAN for illegal moves) - policy distribution for next move
     whiteOwnership (boardXSize * boardYSize floats) - predicted ownership by white (from -1 to 1).
     ```
     Any consumers of this data should attempt to be robust to any pattern of whitespace within the output, as well as possibly the future addition of new keys and values. The ordering of the keys is also not guaranteed - consumers should be capable of handling any permutation of them.
  * `kata-get-param PARAM`, `kata-set-param PARAM VALUE`
     * Get a parameter or set a parameter to a given value.
     * Currently, the only supported PARAM is `playoutDoublingAdvantage (float)`. More may be added later.
