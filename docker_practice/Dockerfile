FROM python:3.8-slim

MAINTAINER hotaiconnected Jerry.Ko

# Install manually all the missing libraries
RUN apt-get update
RUN apt-get install -y \
	pip \
	gconf-service \
	libasound2 \
	libatk1.0-0 \
	libcairo2 \
	libcups2 \
	libfontconfig1 \
	libgdk-pixbuf2.0-0 \
	libgtk-3-0 \
	libnspr4 \
	libpango-1.0-0 \
	libxss1 \
	fonts-liberation \
	libnss3 \
	lsb-release \
	xdg-utils
RUN apt-get install -y libappindicator1; apt-get -fy install

# Install Python dependencies.
COPY requirements.txt \tmp\requirements.txt
RUN pip install -r \tmp\requirements.txt

COPY init_pyppeteer.py \tmp\init_pyppeteer.py
RUN python3 \tmp\init_pyppeteer.py
COPY main.py main.py

EXPOSE 5000

CMD python main.py