 yii - catch and log MySQL deadlock errors
===========================================

This method allows to log InnoDB monitor output when deadlock error occured.
This way we will have much more useful data to find and fix deadlock.
Extend error handler class:

    class AppErrorHandler extends CErrorHandler {
        protected function handleException($exception) {
            /* CDbCommand failed to execute the SQL statement: SQLSTATE[40001]:
            * Serialization failure: 1213 Deadlock found when trying to get lock;
            * try restarting transaction. The SQL statement executed was:
            * INSERT INTO `table_name` (`id`, `name`) VALUES (:yp0, :yp1)
            */
            //can we check $exception->getCode() ?
            if ($exception instanceof CDbException 
                && strpos($exception->getMessage(), 'Deadlock') !== false
            ) {
                $data = Yii::app()->db->createCommand('SHOW ENGINE INNODB STATUS')->query();
                $info = $data->read();
                $info = serialize($info);
                Yii::log('Deadlock error, innodb status: ' . $info,
                    CLogger::LEVEL_ERROR,'system.db.CDbCommand');
            }
            return parent::handleException($exception);
        }
    }

Place it in `application/protected/components` and set in the `config/main.php`:

    return array(
        ...
        'components' => array(
            'errorHandler' => array(
                'class' => 'AppErrorHandler',
            ),
        ),
        ...
    );

Links
------------------------

[InnoDB Lock Modes](http://dev.mysql.com/doc/refman/5.0/en/innodb-lock-modes.html)

[InnoDB Record, Gap, and Next-Key Locks](http://dev.mysql.com/doc/refman/5.0/en/innodb-record-level-locks.html)

[MySQL Forums :: Transactions :: Deadlock With DELETE/INSERT Queries](http://forums.mysql.com/read.php?97,79242,79242#msg-79242)

[MySQL Forums :: InnoDB :: Explain deadlock locking the same index](http://forums.mysql.com/read.php?22,288541,288782#msg-288782)

