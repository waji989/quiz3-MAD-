import 'package:flutter/material.dart';

void main() => runApp(FlashCardApp());

class FlashCardApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Flashcards Quiz',
      theme: ThemeData(primarySwatch: Colors.deepPurple),
      home: FlashCardHome(),
    );
  }
}

class FlashCard {
  final String question;
  final String answer;
  bool learned;

  FlashCard({required this.question, required this.answer, this.learned = false});
}

class FlashCardHome extends StatefulWidget {
  @override
  State<FlashCardHome> createState() => _FlashCardHomeState();
}

class _FlashCardHomeState extends State<FlashCardHome> {
  final GlobalKey<AnimatedListState> _listKey = GlobalKey<AnimatedListState>();
  List<FlashCard> cards = [
    FlashCard(question: 'What is Flutter?', answer: 'An open-source UI toolkit by Google.'),
    FlashCard(question: 'What language does Flutter use?', answer: 'Dart.'),
    FlashCard(question: 'What is a Widget?', answer: 'A basic building block of Flutter UI.'),
    FlashCard(question: 'What is setState()?', answer: 'A method to rebuild the widget tree.'),
    FlashCard(question: 'What is hot reload?', answer: 'A feature to update code instantly.'),
  ];

  Future<void> _refreshCards() async {
    await Future.delayed(const Duration(seconds: 1));
    setState(() {
      cards.shuffle();
    });
  }

  void _addNewCard() {
    final newCard = FlashCard(
      question: 'New Question ${cards.length + 1}',
      answer: 'New Answer ${cards.length + 1}',
    );
    cards.insert(0, newCard);
    _listKey.currentState?.insertItem(0);
  }

  int get learnedCount => cards.where((c) => c.learned).length;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      floatingActionButton: FloatingActionButton(
        onPressed: _addNewCard,
        child: const Icon(Icons.add),
      ),
      body: RefreshIndicator(
        onRefresh: _refreshCards,
        child: CustomScrollView(
          slivers: [
            SliverAppBar(
              pinned: true,
              expandedHeight: 120,
              flexibleSpace: FlexibleSpaceBar(
                title: Text('Learned $learnedCount of ${cards.length}'),
                background: Container(color: Colors.deepPurpleAccent),
              ),
            ),
            SliverToBoxAdapter(
              child: AnimatedList(
                key: _listKey,
                shrinkWrap: true,
                physics: const NeverScrollableScrollPhysics(),
                initialItemCount: cards.length,
                itemBuilder: (context, index, animation) {
                  final card = cards[index];
                  return SizeTransition(
                    sizeFactor: animation,
                    child: Dismissible(
                      key: ValueKey(card.question),
                      onDismissed: (_) {
                        setState(() {
                          card.learned = true;
                          cards.removeAt(index);
                        });
                      },
                      background: Container(
                        color: Colors.green,
                        alignment: Alignment.centerLeft,
                        padding: const EdgeInsets.only(left: 20),
                        child: const Icon(Icons.check, color: Colors.white),
                      ),
                      child: FlashCardTile(card: card),
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class FlashCardTile extends StatefulWidget {
  final FlashCard card;
  const FlashCardTile({required this.card});

  @override
  State<FlashCardTile> createState() => _FlashCardTileState();
}

class _FlashCardTileState extends State<FlashCardTile> {
  bool _showAnswer = false;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      elevation: 4,
      child: ListTile(
        title: Text(
          _showAnswer ? widget.card.answer : widget.card.question,
          style: TextStyle(
              fontWeight: FontWeight.bold,
              color: _showAnswer ? Colors.green : Colors.black87),
        ),
        onTap: () {
          setState(() {
            _showAnswer = !_showAnswer;
          });
        },
      ),
    );
  }
}
