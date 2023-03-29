# Алексей Зеленков
# Группа an-35 (Android-разработчик с нуля)

# HomeWork32

## Вопросы: Cancellation

Вопрос 1. Отработает ли в этом коде строка <--? Поясните, почему да или нет.


    fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        launch {
            delay(500)
            println("ok1") // <--
        }
        launch {
            delay(500)
            println("ok2")
        }
    }
    delay(100)
    job.cancelAndJoin()
    }
## Ответ: строка не отработает. Она выполняется через 500 мс, но уже через 100 мс материнская корутина даст команду на прерывание job


Вопрос 2. Отработает ли в этом коде строка <--? Поясните, почему да или нет.

    fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        val child = launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
        delay(100)
        child.cancel()
    }
    delay(100)
    job.join()
    }

## Ответ: строка не отработает. Она выполняется через 500 мс, но уже через 100 мс материнская корутина даст команду на прерывание child



## Вопросы: Exception Handling

## Вопрос 1. Отработает ли в этом коде строка <--? Поясните, почему да или нет.

    fun main() {
    with(CoroutineScope(EmptyCoroutineContext)) {
        try {
            launch {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
    }

## Ответ: корутина запускается внутри "try". Исключение "something bad happened" не обрабатывается, значит, корутина упадет. А за ней по иерархии упадет и весь scope. До "catch" дело не дойдет, строка "<--" не выполнится. Поток main завершится штатно.



## Вопрос 2. Отработает ли в этом коде строка <--? Поясните, почему да или нет.
    fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
    }
## Ответ: мы создаем область видимости корутин, но корутину внутри нее не создаем. Сразу выбрасываем исключение. Я так понял, coroutineScope не распространяет исключение по иерархии. Поэтому catch отлавливает исключение, и строка "<--" сработает.


## Вопрос 3. Отработает ли в этом коде строка <--? Поясните, почему да или нет.
    fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
    }
## Ответ: создается supervisorScope без корутин. catch отловит его и строка "<--" выполнится


## Вопрос 4. Отработает ли в этом коде строка <--? Поясните, почему да или нет.
    fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
    Thread.sleep(1000)
    }

## Ответ: пока первая корутина ждет 500 мс, вторая сгенерирует исключение. coroutineScope перехватит его и отменит выполнение всего скоупа. строка "<--" не выполнится


## Вопрос 5. Отработает ли в этом коде строка <--? Поясните, почему да или нет.
    fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
    }
## Ответ: supervisorScope позволяет выполниться всем корутинам в скоупе. Сначала вторая корутина сгенерирует исключение. Затем, через 500 мс, первая корутина также сгенерирует исключение. Так как supervisorScope сам обрабатывает ошибки, catch их не увидит, строка "<--" не выполнится.


## Вопрос 6. Отработает ли в этом коде строка <--? Поясните, почему да или нет.
    fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
    }
## Ответ: пока обе корутины отрабатывают свои delay, сгенерируется исключение и весь скоуп прервется. Cтрока "<--" не выполнится


## Вопрос 7. Отработает ли в этом коде строка <--? Поясните, почему да или нет.
    fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext + SupervisorJob()).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
    }
## Ответ: строка "<--" не запустится, так как исключение прерывает материнскую корутину
