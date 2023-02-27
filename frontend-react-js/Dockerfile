FROM node:16.18 AS builder

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
#EXPOSE ${PORT}
CMD ["npm", "start"]

FROM alpine:latest
EXPOSE ${PORT}
WORKDIR /frontend-react-js
RUN apk --no-cache add ca-certificates
COPY --from=builder /frontend-react-js /frontend-react-js/multistage
CMD ["./multistage"]

