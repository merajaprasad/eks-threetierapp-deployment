# Stage 1: Build
FROM node:14 as builder

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json for dependency installation
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application files
COPY . .

# Build the application (if applicable, e.g., for React or Angular)
# RUN npm run build

# Stage 2: Production Image
FROM node:14-slim

# Set the working directory in the final image
WORKDIR /usr/src/app

# Copy only the necessary files from the builder stage
COPY --from=builder /usr/src/app .

# Expose the port that the app runs on
EXPOSE 3000

# Run the application
CMD [ "npm", "start" ]

