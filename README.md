#include <QApplication>
#include <QWidget>
#include <QPushButton>
#include <QLineEdit>
#include <QLabel>
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QListWidget>
#include <QInputDialog>
#include <vector>
#include <QString>

class RouletteTracker : public QWidget {
    Q_OBJECT

private:
    QLineEdit *bidInput;
    QLineEdit *spinInput;
    QLabel *bidLabel;
    QListWidget *poolDisplay;
    std::vector<int> numbersPool;
    double currentBid;

public:
    RouletteTracker(QWidget *parent = nullptr) : QWidget(parent), currentBid(0.0) {
        // Widgets
        QLabel *bidPrompt = new QLabel("Enter your initial bid in LEI:", this);
        bidInput = new QLineEdit(this);

        QPushButton *setBidButton = new QPushButton("Set Initial Bid", this);
        connect(setBidButton, &QPushButton::clicked, this, &RouletteTracker::setInitialBid);

        QLabel *spinPrompt = new QLabel("Enter the result of the current spin:", this);
        spinInput = new QLineEdit(this);

        QPushButton *addSpinButton = new QPushButton("Add Spin Result", this);
        connect(addSpinButton, &QPushButton::clicked, this, &RouletteTracker::addSpinResult);

        QPushButton *showPoolButton = new QPushButton("Show Current Pool", this);
        connect(showPoolButton, &QPushButton::clicked, this, &RouletteTracker::showCurrentPool);

        QPushButton *exitButton = new QPushButton("Exit", this);
        connect(exitButton, &QPushButton::clicked, this, &QWidget::close);

        bidLabel = new QLabel("Current Bid: 0 LEI", this);
        poolDisplay = new QListWidget(this);

        // Layouts
        QVBoxLayout *mainLayout = new QVBoxLayout(this);
        QHBoxLayout *bidLayout = new QHBoxLayout();
        QHBoxLayout *spinLayout = new QHBoxLayout();

        bidLayout->addWidget(bidPrompt);
        bidLayout->addWidget(bidInput);
        bidLayout->addWidget(setBidButton);

        spinLayout->addWidget(spinPrompt);
        spinLayout->addWidget(spinInput);
        spinLayout->addWidget(addSpinButton);

        mainLayout->addLayout(bidLayout);
        mainLayout->addWidget(bidLabel);
        mainLayout->addLayout(spinLayout);
        mainLayout->addWidget(showPoolButton);
        mainLayout->addWidget(poolDisplay);
        mainLayout->addWidget(exitButton);

        setLayout(mainLayout);
    }

private slots:
    void setInitialBid() {
        bool ok;
        double bid = bidInput->text().toDouble(&ok);
        if (ok && bid > 0) {
            currentBid = bid;
            bidLabel->setText("Current Bid: " + QString::number(currentBid) + " LEI");
        } else {
            bidLabel->setText("Invalid bid. Please enter a valid amount.");
        }
    }

    void addSpinResult() {
        bool ok;
        int spin = spinInput->text().toInt(&ok);
        if (ok) {
            numbersPool.push_back(spin);
            if (numbersPool.size() > 20) {
                numbersPool.erase(numbersPool.begin());
            }

            bool outcomeOk;
            QString outcome = QInputDialog::getText(this, "Spin Outcome", "Did you win this spin? (y/n):", QLineEdit::Normal, "", &outcomeOk);

            if (outcomeOk && outcome.toLower() == "y") {
                bidLabel->setText("You won! Current Bid remains: " + QString::number(currentBid) + " LEI");
            } else if (outcomeOk && outcome.toLower() == "n") {
                currentBid *= 2;
                bidLabel->setText("You lost! New Bid: " + QString::number(currentBid) + " LEI");
            } else {
                bidLabel->setText("Invalid input. Spin result not recorded.");
            }
        } else {
            bidLabel->setText("Invalid spin result. Please enter a valid number.");
        }
    }

    void showCurrentPool() {
        poolDisplay->clear();
        for (int num : numbersPool) {
            poolDisplay->addItem(QString::number(num));
        }
    }
};

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    RouletteTracker tracker;
    tracker.setWindowTitle("Roulette Tracker");
    tracker.resize(400, 300);
    tracker.show();

    return app.exec();
}
