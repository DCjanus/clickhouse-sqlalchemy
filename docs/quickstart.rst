.. _quickstart:

Quickstart
==========

This page gives a good introduction to clickhouse-sqlalchemy.
It assumes you already have clickhouse-sqlalchemy installed.
If you do not, head over to the :ref:`installation` section.

It should be pointed that session must be created with
``clickhouse_sqlalchemy.make_session``. Otherwise ``session.query`` and
``session.execute`` will not have ClickHouse SQL extensions. The same is
applied to ``Table`` and ``get_declarative_base``.


Let's define some table, insert data into it and query inserted data.

    .. code-block:: python

        from typing import Optional
        from datetime import date, timedelta
        from sqlalchemy import create_engine, Column, MetaData, select, func
        from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, Session
        from sqlalchemy.engine import URL

        from clickhouse_sqlalchemy import (
            Table, make_session, get_declarative_base, types, engines
        )

        # Create the connection URL
        url = URL.create(
            drivername="clickhouse+native",
            host="localhost",
            database="default"
        )

        # Create engine and metadata
        engine = create_engine(url)
        metadata = MetaData()

        # Create base class for declarative models
        Base = get_declarative_base(metadata=metadata)

        # Define the model with type annotations
        class Rate(Base):
            __tablename__ = "rate"

            day: Mapped[date] = mapped_column(types.Date, primary_key=True)
            value: Mapped[int] = mapped_column(types.Int32)

            __table_args__ = (
                engines.Memory(),
            )

        # Create the table
        metadata.create_all(engine)


Now it's time to insert some data and query it using SQLAlchemy 2.0 style:

    .. code-block:: python

        # Create data to insert
        today = date.today()
        rates = [
            {"day": today - timedelta(i), "value": 200 - i}
            for i in range(100)
        ]

        # Using a session with context manager
        with Session(engine) as session:
            # Insert data
            session.execute(Rate.__table__.insert(), rates)
            session.commit()

            # Query data using 2.0 style
            stmt = (
                select(func.count())
                .select_from(Rate)
                .where(Rate.day > today - timedelta(20))
            )
            count = session.scalar(stmt)
            print(f"Found {count} records")

Now you are ready to :ref:`configure your connection<connection>` and see more
ClickHouse :ref:`features<features>` support.