# はじめに

QGIS では、python を使用して処理に時間がかかるタスクを実行する場合、画面が固まってしまいます。画面が固まることでユーザは QGIS を操作することができなくなります。

QGIS では、QgsTask を使用することで、タスクを非同期で実行することができます。この記事では、QgsTask を使用して QGIS をフリーズさせずに python を実行する方法を紹介します。

## 1. 画面が固まってしまう例

試しに、以下のコードを QGIS の python コンソールに貼り付けて実行してみてください。画面が固まってしまうことが確認できると思います。

```python
from time import sleep

for i in range(50):
    print(i)
    sleep(0.2)
```

## 2. QgsTask を使用して非同期でタスクを実行する

```python
from qgis.core import Qgis, QgsApplication, QgsMessageLog, QgsTask
from time import sleep

class PrintTask(QgsTask):
    def __init__(self, description, number):
        super().__init__(description)
        self.description = description
        self.number = number

    def run(self):
        try:
            for i in range(self.number):
                print(i)
                sleep(0.2)
        except:
            self.exception = Exception('bad value')
            return False

        return True

    def finished(self, result):
        if result:
            QgsMessageLog.logMessage(
                "Completed!!",
                "PrintTask",
                Qgis.Success
            )
        else:
            QgsMessageLog.logMessage(
                "Error!!",
                "PrintTask",
                Qgis.Critical
            )
            raise self.exception

    def cancel(self):
        QgsMessageLog.logMessage(
            f'PrintTask {self.description} was canceled',
            "PrintTask",
            Qgis.Info
        )
        super().cancel()


task = PrintTask("5000 task", 500)
QgsApplication.taskManager().addTask(task)
```
